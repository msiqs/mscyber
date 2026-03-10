# Interceptação de Tráfego de Rede

A maioria dos apps modernos se comunica com uma API backend via HTTPS. Interceptar esse tráfego permite ler e modificar requisições antes que cheguem ao servidor, expondo falhas que afetam todos os usuários da plataforma.

## 1. Proxying com Burp Suite

O objetivo é posicionar o Burp Suite entre o app e o servidor:

**Fluxo normal:** `App` -> `Servidor`

**Fluxo com proxy:** `App` -> `Burp Suite` -> `Servidor`

Com o Burp interceptando, é possível:
- **Ler:** dados em texto claro (JSON, XML, tokens)
- **Modificar:** parâmetros antes que o servidor os receba (ex: trocar `id=10` por `id=1`)

## 2. HTTPS e o Problema do Certificado

O Burp usa um certificado CA próprio para "abrir" e "fechar" o TLS. O Android precisa confiar nesse certificado, caso contrário o app recusa a conexão com `SSLHandshakeException`.

**Antes do Android 7:** bastava instalar o certificado CA do Burp nas configurações de usuário.

**Android 7+:** apps ignoram certificados de usuário por padrão e só confiam em certificados do sistema, instalados em `/system/etc/security/cacerts`.

**Solução com root:**
1. Exportar o certificado CA do Burp Suite
2. Mover o arquivo para `/system/etc/security/cacerts/` via ADB ou Magisk
3. Reiniciar o dispositivo

## 3. SSL Pinning

Mesmo com o certificado instalado no sistema, apps com SSL Pinning armazenam internamente o hash do certificado esperado do servidor. Se o certificado recebido não bater com o armazenado, a conexão é encerrada.

**Bypass:** hookar via Frida a função de validação do certificado e forçá-la a aceitar qualquer entrada.

> Combo clássico: Frida (bypass do pinning) + Burp Suite (interceptação e manipulação).

### Network Security Config

Antes de usar o Frida, verifique `res/xml/network_security_config.xml` na análise estática:

- `cleartextTrafficPermitted="true"`: o app aceita HTTP, sem necessidade de TLS bypass
- `<pin-set>` com hashes definidos: pinning ativo, requer Frida para bypass

### Flutter e Xamarin

Frameworks cross-platform (Flutter, Xamarin) não usam o proxy do sistema Android nem respeitam a TrustStore nativa. O Burp sozinho não funciona.

**Solução:** scripts Frida específicos que hookam a biblioteca `libflutter.so` e desabilitam a verificação de certificado diretamente na engine BoringSSL.

## 4. O Que Buscar no Tráfego

Com o tráfego fluindo no Burp, os principais vetores a explorar:

| Vulnerabilidade | Técnica |
|---|---|
| IDOR | Trocar o ID na URL: `/api/users/1234` -> `/api/users/1` |
| Mass Assignment | Injetar campos extras no JSON: `"is_admin": true` |
| Vazamento de dados | Comparar o que o app exibe com o que o JSON retorna |

## Kill Chain Resumida

1. **ADB:** conecta ao dispositivo
2. **Root:** permite modificar o sistema
3. **Certificado de sistema:** habilita o MitM básico
4. **Frida:** desativa o SSL Pinning
5. **Burp Suite:** intercepta e explora a API
