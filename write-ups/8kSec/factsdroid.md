# Write-Up: FactsDroid Subversão Criptográfica e Sequestro de Rede em Arquitetura Flutter

## 1. Escopo e Reconhecimento Arquitetural

**Alvo:** Aplicativo FactsDroid
**Ambiente:** Emulador Android (x86_64)
**Framework:** Flutter (Dart VM 3.7.2)

Aplicações Flutter compiladas via **AOT (Ahead-of-Time)** apresentam uma superfície de ataque fundamentalmente distinta do ecossistema Android nativo (Java/Kotlin). Os pontos críticos identificados foram:

| Característica | Implicação Ofensiva |
|---|---|
| **Rede Autárquica** | O motor nativo (`libflutter.so`) ignora ativamente as configurações de proxy da Camada 7 do Android |
| **Criptografia Estática** | A validação X.509 não usa a Trust Store do sistema é gerenciada em memória pelo BoringSSL embarcado |
| **Stripped Binaries** | Ausência de tabela de símbolos; exige RE focada em padrões de Assembly e XREFs |

**Objetivo:** Anular as defesas criptográficas (SSL Pinning) e as políticas de roteamento para expor a comunicação com a API backend em texto claro.

---

## 2. Fase 1 Anulação de Telemetria Defensiva (RASP)

Durante a inicialização a frio, a auditoria do `logcat` e do SELinux revelou mecanismos de **Anti-Emulation/RASP**. O aplicativo (PID 6637) instanciou subprocessos para ler propriedades do host, invocando `/system/bin/getprop` para extrair a flag `ro.debuggable`.

### Vetor de Ataque Camuflagem Ambiental via Frida

Dois pontos de interceptação foram instrumentados dinamicamente:

- **`android.os.SystemProperties`** retorno forjado das propriedades críticas:
  - `ro.debuggable = 0`
  - `ro.secure = 1`

- **`java.lang.Runtime.exec`** filtragem e bloqueio de subprocessos contendo a string `getprop`, redirecionando o fluxo para comandos inofensivos e cegando a telemetria defensiva.

---

## 3. Fase 2 Mapeamento de Memória e Engenharia Reversa (BoringSSL)

A tentativa inicial de interceptação de tráfego resultou no encerramento da conexão TLS com o seguinte artefato no console Dart:

```
CERTIFICATE_VERIFY_FAILED: unable to get local issuer certificate(handshake.cc:391)
```

O erro apontava para a **linha 391** (`0x187` em hexadecimal) do código-fonte C++ `handshake.cc` do BoringSSL âncora para o ataque em nível de memória.

### Dissecção Estática no Ghidra

1. `libflutter.so` carregado no Ghidra.
2. String `"../../../flutter/third_party/boringssl/src/ssl/handshake.cc"` localizada em `.rodata`.
3. XREFs traçados até `.text`, revelando múltiplas funções despachantes.
4. Pseudocódigo (Decompile) identificou `FUN_008c4c99` função que continha a lógica de falha, acionando `ERR_put_error` com o parâmetro de linha `0x187`.

**Semântica da função:** máquina de estados de validação do certificado par.
- Retorno `0` → sucesso (handshake TLS continua)
- Retorno `!= 0` → falha (conexão bloqueada)

### Cálculo do Offset

```
0x008c4c99  (endereço absoluto no Ghidra)
- 0x100000  (Image Base)
─────────────────────────────────────
= 0x7C4C99  (offset definitivo em libflutter.so)
```

---

## 4. Fase 3 Amputação de Rotina e Subversão Criptográfica

Interceptações condicionais (modificar o registrador de retorno apenas no momento do erro) provaram-se instáveis, gerando **corrupção da struct SSL** e crashes do tipo `SIGSEGV` (Null Pointer Dereference) por condições de corrida em operações I/O assíncronas (`WANT_READ`).

### A Manobra Substituição Ativa via `Interceptor.replace`

A estratégia evoluiu de interceptação passiva para **substituição ativa**:

1. A rotina original no offset `0x7C4C99` foi **integralmente amputada** do fluxo de execução.
2. Uma **`NativeCallback`** foi injetada em seu lugar via Frida.
3. Sempre que o motor Flutter solicitava verificação do certificado X.509 do proxy, a função reescrita retornava instantaneamente `0` forçando confiança absoluta no túnel.

**Detalhe operacional:** A injeção foi ancorada via hook em `io.flutter.embedding.engine.FlutterJNI.loadLibrary(Context)`, garantindo que o isolamento de namespace do Android Bionic não fosse violado e prevenindo `UnsatisfiedLinkError`.

---

## 5. Fase 4 Sequestro de Rota na Camada de Rede (Netfilter)

Com a proteção criptográfica destruída, restou contornar a **evasão de proxy nativa do Flutter**: a Dart VM não obedece às configurações de proxy do Wi-Fi do sistema.

### Solução Redirecionamento via iptables (NAT/Netfilter)

Regras de NAT injetadas no kernel do Android para forçar todo tráfego de saída para o listener da máquina atacante:

```bash
iptables -t nat -F
iptables -t nat -A OUTPUT -p tcp --dport 80  -j DNAT --to-destination 10.0.2.2:8080
iptables -t nat -A OUTPUT -p tcp --dport 443 -j DNAT --to-destination 10.0.2.2:8080
```

**Configuração do Burp Suite:**
- Listener em `All Interfaces`
- Opção `Support invisible proxying` ativada

O tráfego do aplicativo perdeu a capacidade de alcançar a internet diretamente e desaguou em **texto claro** no proxy, abrindo o vetor para exploração na Camada 7 e dissecção completa da API do FactsDroid.

---

## Resumo do Ataque

```
[RASP Bypass]          Frida hook em SystemProperties + Runtime.exec
      |
      v
[BoringSSL RE]         Ghidra: XREF handshake.cc:391 → FUN_008c4c99 → offset 0x7C4C99
      |
      v
[SSL Pinning Kill]     Interceptor.replace → NativeCallback retorna 0
      |
      v
[Proxy Redirect]       iptables DNAT 443/80 → 10.0.2.2:8080 (Burp)
      |
      v
[API Exposed]          Tráfego em texto claro disponível para análise
```
