# ADB (Android Debug Bridge)

O ADB é a ferramenta principal para interagir com dispositivos Android via terminal. Ele cria uma ponte entre o computador e o dispositivo (emulador ou físico via USB), sendo indispensável para pentest mobile, sem ele, ferramentas como Frida e Drozer não funcionam.

---

## Comandos Essenciais

### `adb shell`
Abre um terminal remoto no Android, com acesso ao sistema de arquivos e execução de comandos.

```bash
adb shell
```

> Ponto de partida para navegar em `/data/data/<pacote>/` e inspecionar arquivos deixados sem proteção.

---

### `adb pull` / `adb push`

Transfere arquivos entre o dispositivo e o computador.

```bash
# Extrair arquivo do dispositivo
adb pull /data/data/com.app.alvo/databases/app.db ./

# Enviar arquivo para o dispositivo
adb push frida-server /data/local/tmp/
```

---

### `adb logcat`

Exibe os logs do sistema em tempo real. Desenvolvedores frequentemente esquecem de desativar logs em produção.

```bash
adb logcat | grep -i "token\|password\|key\|secret"
```

> É comum encontrar tokens, senhas e chaves de API expostos nos logs enquanto o app está em uso.

---

### `am` e `pm` (dentro do `adb shell`)

Ferramentas para interagir com componentes do Android:

**Package Manager (`pm`)**, lista pacotes instalados:
```bash
pm list packages -f | grep alvo
```

**Activity Manager (`am`)**, dispara componentes:
```bash
# Forçar abertura de uma Activity exportada
am start -n com.app.alvo/.ActivityAdmin
```

> Se uma Activity estiver exportada sem proteção, pode ser acessada diretamente via `am`, sem precisar criar um app malicioso.

---

### `adb forward`

Cria um túnel TCP entre o computador e o dispositivo via USB.

```bash
adb forward tcp:27042 tcp:27042
```

> Necessário para conectar ferramentas como o Frida ao agente rodando no dispositivo.

---

### `adb backup`

Extrai um backup do sandbox do app (quando permitido pelo manifest).

```bash
adb backup -f backup.ab com.app.alvo
```

> Se o app tiver `android:allowBackup="true"`, o arquivo `.ab` pode ser convertido para `.tar` e conterá bancos de dados e preferências, sem necessidade de root.
