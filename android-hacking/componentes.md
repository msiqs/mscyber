# Componentes do Android

Diferente de um programa desktop com um único ponto de entrada (`main()`), um app Android é composto por componentes distintos que podem ser invocados individualmente pelo sistema ou por outros apps. Cada componente é um vetor de ataque em potencial.

O mapa desses componentes fica no `AndroidManifest.xml`, o primeiro arquivo a ser analisado em um pentest.

## Activities

A Activity representa uma tela da interface do usuário.

**Vetor: Authentication Bypass.** Se uma Activity sensível (pós-login, redefinição de senha, painel admin) estiver marcada com `exported="true"`, qualquer app pode forçar sua abertura diretamente, pulando a tela de login.

```bash
# Forçar abertura de Activity exportada via ADB
am start -n com.app.alvo/.AdminActivity
```

## Services

Services executam tarefas em segundo plano sem interface, como sincronização de dados ou downloads.

**Vetor: Privilege Escalation.** Se um serviço privilegiado não verificar a identidade de quem o invoca, um app malicioso pode iniciá-lo e executar ações em nome do usuário (ex: disparar uma transferência bancária silenciosamente).

## Broadcast Receivers

Receivers ficam em espera e reagem a eventos do sistema ou de outros apps (ex: `BOOT_COMPLETED`, `SMS_RECEIVED`).

**Vetores:**
- **Sniffing:** se um app envia um broadcast com dados sensíveis (token de sessão) sem definir permissões, qualquer app pode registrar um Receiver e interceptar esses dados
- **Spoofing:** um atacante envia um broadcast falso para enganar o app alvo e forçá-lo a executar uma ação indesejada

## Content Providers

Gerenciam acesso a dados compartilhados entre apps, geralmente via SQLite (ex: a agenda do sistema compartilhada com o WhatsApp).

**Vetor: SQL Injection.** Se o Provider não sanitizar os inputs recebidos, é possível injetar comandos SQL para extrair o banco de dados completo ou realizar path traversal para leitura de arquivos locais.

## WebViews

Componente que renderiza HTML/JS dentro do app nativo, sem abrir o navegador externo.

**Vetor: JavaScript Bridge (RCE).** Quando o desenvolvedor usa `addJavascriptInterface`, o JavaScript da página ganha acesso a métodos Java nativos. Cenário de ataque:

1. Atacante encontra XSS na página carregada pela WebView
2. O XSS invoca a bridge Java
3. Atacante obtém execução de código nativo no dispositivo

## Intents

Intents são mensagens assíncronas usadas para comunicação entre componentes: "abrir câmera", "iniciar serviço de download", etc.

**Vetor: Intent Injection.** Manipular Intents consiste em fazer o sistema entregar uma mensagem maliciosa a um componente confiável, forçando-o a vazar dados ou executar ações não autorizadas.

### Deep Links

Deep Links permitem que sites ou apps externos disparem Intents dentro de um app via URL (ex: `nullbank://transfer/`).

**Vetor: Mobile CSRF.** Se o app não validar a origem do Deep Link, um link malicioso enviado por SMS ou WhatsApp pode abrir o app bancário da vítima e preencher automaticamente uma tela de transferência.

### PendingIntents

Um PendingIntent é um token criado pelo App A e entregue ao App B, autorizando B a executar uma ação futura com as permissões de A.

**Vetor:** se o App A cria um PendingIntent vazio ou genérico e o passa para um app malicioso, o atacante pode preencher esse Intent com conteúdo arbitrário. Quando o App A executar o Intent, ele estará usando suas próprias permissões contra si mesmo, podendo sobrescrever arquivos ou conceder acesso ao atacante.

> Esse vetor é responsável por vários CVEs relevantes em versões recentes do Android.
