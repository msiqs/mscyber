# ADB (Android Debug Bridge)

Agora que sabemos onde estão as portas de entrada, precisamos de uma ferramenta para interagir com elas. Não usamos o dedo na tela para hackear; usamos o terminal.

A ferramenta fundamental para qualquer pentester mobile é o **ADB (Android Debug Bridge)**. Ele é uma ferramenta de linha de comando que cria uma ponte de comunicação entre o seu computador e o dispositivo Android, seja um emulador ou um celular físico via USB.

Para um atacante, o ADB não é apenas uma ferramenta de debug, é o nosso shell remoto, nosso transportador de arquivos e nosso injetor de comandos. Sem ele, ferramentas avançadas como Frida ou Drozer não funcionam.

## Por que o ADB é vital para o Hacking?

O ADB nos dá acesso direto ao Linux Kernel que roda por baixo da interface bonita do Android. Aqui estão as funções críticas para a segurança ofensiva:

### 1. Acesso ao Shell (`adb shell`)
Este comando abre um terminal remoto dentro do Android. É aqui que exploramos o sistema de arquivos, verificamos permissões de pastas e executamos binários nativos.
* **Uso Ofensivo:** Navegar até `/data/data/com.app.alvo/` para tentar ler bancos de dados ou arquivos de configuração que o desenvolvedor esqueceu de proteger.

### 2. Extração e Injeção (`adb pull` / `adb push`)
* **`adb pull`:** Permite extrair arquivos do dispositivo para o seu computador. Usamos isso para baixar o APK do aplicativo alvo ou para roubar um banco de dados SQLite encontrado.
* **`adb push`:** Envia arquivos do computador para o Android. É assim que enviamos nossos exploits, scripts de enumeração ou o servidor do Frida para dentro do dispositivo.

### 3. O Espião de Logs (`adb logcat`)
O Android possui um sistema de log centralizado. Desenvolvedores usam para debugar erros, mas frequentemente esquecem de desativar logs de produção.
* **Uso Ofensivo:** Deixar o `adb logcat` rodando enquanto usa o aplicativo alvo. É assustadoramente comum ver tokens de autenticação, chaves de API, senhas em texto claro ou URLs sensíveis passando voando pelo log em tempo real. É o equivalente a um sniffing local.

### 4. Manipulação de Componentes (`am` e `pm`)
Lembra dos componentes Activities, Services? Dentro do `adb shell` temos utilitários poderosos para interagir com eles:
* **Package Manager (`pm`):** Lista todos os apps instalados e seus caminhos.
    * *Comando:* `pm list packages -f | grep alvo`
* **Activity Manager (`am`):** A arma para disparar componentes.
    * *Ataque:* Se encontramos uma Activity exportada vulnerável, não precisamos criar um app malicioso para explorá-la. Podemos simplesmente usar o `am` para forçar sua abertura:
    `am start -n com.app.alvo/.ActivitySecretaAdmin`

### 5. Port Forwarding (`adb forward`)
Muitas ferramentas de hacking rodam no computador e precisam falar com um agente dentro do celular.
* **Uso Ofensivo:** O `adb forward` cria um túnel TCP via USB. É essencial para conectar o **Frida** ou para permitir que o navegador do celular acesse um servidor malicioso rodando no seu Localhost.

### 6. Backup Extraction (`adb backup`)
Embora depreciado em versões recentes, muitos apps ainda permitem backup, `android:allowBackup="true"` no manifest.
* **Ataque:** `adb backup com.app.alvo`. Se funcionar, você extrai um arquivo `.ab` contendo todo o sandbox do app. Converta para `.tar` e você terá acesso aos bancos de dados e preferências sem precisar de Root.