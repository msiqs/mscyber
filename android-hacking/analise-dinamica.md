# Análise Dinâmica com Frida

Enquanto a análise estática examina o código em repouso, a análise dinâmica atua com o app em execução, interceptando chamadas, alterando retornos e contornando proteções em tempo real.

A ferramenta central para isso é o **Frida**.

---

## Como o Frida Funciona

O Frida injeta uma engine JavaScript (V8) dentro do processo do app alvo. Você escreve scripts JS no seu computador, e o Frida os executa na memória do Android.

**Dois componentes necessários:**

| Componente | Onde roda | Função |
|---|---|---|
| `frida-server` | Dispositivo Android (root) | Recebe e executa os comandos |
| `frida` (CLI/scripts) | Computador do atacante | Envia scripts e exibe resultados |

```bash
# Enviar o servidor para o dispositivo
adb push frida-server /data/local/tmp/
adb shell "chmod +x /data/local/tmp/frida-server && /data/local/tmp/frida-server &"

# Conectar ao processo alvo
frida -U -n com.app.alvo -l script.js
```

---

## Hooking

A funcionalidade principal do Frida é o **hook**: interceptar uma função e substituir seu comportamento.

**Exemplo: bypassar detecção de root:**

Código original encontrado via JADX:
```java
public boolean isRooted() {
    return new File("/system/bin/su").exists();
}
```

Script Frida para neutralizar a verificação:
```javascript
Java.perform(function() {
    var Security = Java.use("com.app.alvo.SecurityCheck");

    Security.isRooted.implementation = function() {
        return false;
    };
});
```

> O hook substitui a implementação original na memória, sem recompilar o APK, sem quebrar a assinatura.

---

## Casos de Uso

### SSL Pinning Bypass

Apps com SSL Pinning rejeitam certificados que não sejam o seu próprio, bloqueando proxies como o Burp Suite.

**Ataque:** hookar o `TrustManager` ou a biblioteca de rede (ex: OkHttp) para desativar a validação do certificado.

```javascript
// Exemplo genérico de bypass de TrustManager
Java.perform(function() {
    var TrustManager = Java.use("javax.net.ssl.X509TrustManager");
    // hook na implementação customizada do app
});
```

---

### Crypto Hooking

Apps criptografam dados antes de enviá-los à API. Em vez de tentar quebrar a cifra, interceptamos os dados **antes** da criptografia.

**Ataque:** hookar `javax.crypto.Cipher.doFinal()` para capturar o texto plano e a chave no momento da chamada.

```javascript
Java.perform(function() {
    var Cipher = Java.use("javax.crypto.Cipher");

    Cipher.doFinal.overload("[B").implementation = function(data) {
        console.log("Plaintext: " + JSON.stringify(data));
        return this.doFinal(data);
    };
});
```

---

### Trace de Execução

Não sabe quais funções são chamadas ao clicar em um botão?

**Ataque:** usar o `frida-trace` para rastrear todas as chamadas de uma classe em tempo real.

```bash
frida-trace -U -n com.app.alvo -j "com.app.alvo.TransferenciasActivity!*"
```

> Cada clique imprime no terminal as funções acionadas, guiando a engenharia reversa.
