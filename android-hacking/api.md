# Interceptação de Rede: O Ataque ao Backend

Até agora, focamos em explorar o aplicativo que roda no celular (Client-Side). Porém, a maioria dos aplicativos modernos funciona como um navegador glorificado: eles enviam e recebem dados de uma API (Server-Side).

Se você encontrar uma falha no código Java do app, você compromete o dispositivo do usuário. Mas se você interceptar o tráfego e encontrar uma falha na API, você pode comprometer **o servidor da empresa** e todos os seus usuários.

Para fazer isso, precisamos realizar um ataque de **Man-in-the-Middle**.

## 1. O Conceito de Proxying

Normalmente, o App fala diretamente com a API via HTTPS. Para ler essa conversa, precisamos colocar o **Burp Suite**, nosso proxy de interceptação, no meio do caminho.

**Fluxo Normal:** `App` -> `Internet` -> `Servidor`
**Fluxo de Ataque:** `App` -> `Burp Suite (PC)` -> `Internet` -> `Servidor`

Ao configurar o proxy, o Burp Suite "pausa" cada requisição HTTP que sai do celular. Isso nos permite:
* **Ler:** Ver senhas e dados sensíveis em texto claro (JSON/XML).
* **Modificar:** Alterar valores (ex: mudar `id_usuario=10` para `id_usuario=1` ou `valor=100` para `valor=0.01`) antes que o servidor receba o pedido.

## 2. A Barreira: HTTPS e Certificados

Se fosse apenas HTTP, seria fácil. Mas o tráfego é criptografado (HTTPS/TLS). Para o Burp Suite ler o tráfego, ele precisa "abrir" a criptografia e fechá-la novamente para entregar ao servidor.

Para fazer isso, o Burp gera um certificado digital falso (CA - Certificate Authority).
* **O Problema:** O Android não confia em certificados estranhos. Se você tentar interceptar o tráfego sem configuração, o app vai dar erro de conexão (`SSLHandshakeException`), porque ele percebe que tem um "espião" o Burp no meio.

### User vs. System Certificates (A Mudança do Android 7+)

Antigamente, bastava instalar o certificado do Burp nas configurações do usuário.
Desde o Android 7.0, **os aplicativos ignoram certificados de usuário** por padrão. Eles só confiam em certificados instalados na partição do sistema `/system/etc/security/cacerts`.

**A Solução:**
Com **Root**, podemos forçar a instalação do certificado do Burp como se fosse um certificado de sistema.
1.  Exportamos o certificado do Burp.
2.  Usamos o Magisk ou comandos ADB para mover esse arquivo para a pasta do sistema.
3.  Reiniciamos o celular.
Agora, o Android acredita que o Burp Suite é uma entidade confiável, tão segura quanto a Google ou a Verisign.

## 3. SSL Pinning: O Chefão Final

Mesmo com o certificado no sistema, muitos apps implementam uma defesa extra chamada **SSL Pinning**.

O App diz: *"Eu não confio apenas no Sistema. Eu tenho o desenho exato do certificado do meu servidor guardado no meu código. Se o certificado que eu receber for diferente, mesmo que o sistema diga que é confiável, eu corto a conexão."*

É aqui que nossas ferramentas se unem:
1.  **Burp Suite:** Está pronto para ouvir, mas é bloqueado pelo Pinning.
2.  **Frida:** Entra em ação. Usamos um script do Frida para hookar a função de comparação de certificados do app e forçá-la a aceitar qualquer coisa.

> **O Combo Clássico:**
> Frida + Burp Suite, para hooking de métodos e manipulação de requisições respectivamente.

## 4. O Que Procurar no Tráfego?

Uma vez que o tráfego está fluindo no Burp, o que buscamos?

* **IDOR (Insecure Direct Object Reference):** Mudar o ID do usuário na URL `/api/users/1234` para ver dados de outra pessoa.
* **Mass Assignment:** Tentar enviar parâmetros que não estavam na tela original (ex: adicionar `"is_admin": true` no JSON de cadastro).
* **Vazamento de Dados:** Respostas da API que trazem dados demais (ex: o app mostra só o nome, mas o JSON traz o CPF, endereço e saldo).

---

## Resumo da Kill Chain

1.  **ADB:** Conecta no celular.
2.  **Root:** Nos dá permissão para modificar o sistema.
3.  **Certificado de Sistema:** Permite o MitM básico.
4.  **Frida:** Desliga o SSL Pinning.
5.  **Burp Suite:** Intercepta e explora a API.