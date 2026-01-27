# Os Componentes do Android: As Portas de Entrada

Até agora, falamos do Sistema Operacional. Mas nós raramente atacamos o SO diretamente, a menos que tenhamos um 0-day de Kernel. Nós atacamos os aplicativos.

Diferente de um programa desktop tradicional que tem um único ponto de entrada, a função `main()`, um app Android é composto por componentes distintos que podem ser invocados individualmente pelo sistema ou por outros apps. Para um atacante, cada componente é uma potencial porta de entrada para injeção de código, roubo de dados ou bypass de autenticação.

O mapa desses componentes fica no arquivo `AndroidManifest.xml`. É o primeiro lugar que um pentester olha.

## 1. Activities

A **Activity** é a interface do usuário. É cada tela que você vê.

* **Função:** Interagir com o usuário.
* **Vetor de Ataque:** **Exported Activities**. Se um desenvolvedor marcar uma Activity sensível, como a "Tela Pós-Login", "Redefinir Senha" ou "Configurações Administrativas", como `exported="true"`, um atacante pode forçar o aplicativo a abrir essa tela diretamente através de um comando simples, pulando completamente a tela de login. É o clássico "Authentication Bypass".

## 2. Services

Os **Services** rodam em segundo plano, sem interface. Eles tocam música, baixam arquivos ou sincronizam dados enquanto você usa outro app.

* **Função:** Processamento pesado em background.
* **Vetor de Ataque:** **Privilege Escalation**. Um serviço vulnerável pode ser iniciado por um app malicioso para realizar ações privilegiadas em nome da vítima. Imagine um serviço bancário que transfere dinheiro. Se ele não verificar as permissões de quem o chamou, qualquer app instalado no celular, até uma calculadora maliciosa, pode invocar esse serviço e realizar a transação silenciosamente.

## 3. Broadcast Receivers

Eles são os "ouvidos" do aplicativo. Eles ficam dormindo e só acordam quando um evento específico acontece, o sistema avisa "Bateria Baixa", "SMS Recebido" ou "Boot Completado".

* **Função:** Reagir a eventos do sistema ou de outros apps.
* **Vetor de Ataque:**
    1.  **Sniffing:** Se um app envia um broadcast contendo dados sensíveis (token de sessão) e não define permissões, qualquer app malicioso pode registrar um Receiver e interceptar esse dado.
    2.  **Spoofing:** Um atacante pode enviar um broadcast falso "Simule que um SMS do banco chegou" para enganar o app alvo e fazê-lo executar uma ação indesejada, como phishing ou execução de tarefa interna.

## 4. Content Providers

Eles gerenciam o acesso a um repositório central de dados. É a forma padrão de compartilhar dados entre aplicativos (ex: seus contatos do WhatsApp vêm da agenda do telefone via Content Provider).

* **Função:** Abstração de banco de dados, geralmente SQLite.
* **Vetor de Ataque:** **SQL Injection**. Se o Content Provider não sanitizar os inputs que vem de outro app, podemos injetar comandos SQL para extrair o banco de dados inteiro do aplicativo, ler arquivos locais `Path Traversal` ou modificar informações críticas.

## 5. WebViews

Muitos apps modernos são híbridos: uma casca nativa que carrega conteúdo web. A **WebView** é o componente que renderiza páginas HTML/JS dentro do app.

* **Função:** Mostrar conteúdo web sem abrir o Chrome externo.
* **Vetor de Ataque:** **JavaScript Bridges**. O perigo real surge quando o desenvolvedor usa métodos como `addJavascriptInterface`. Isso cria uma ponte onde o JavaScript de uma página consegue executar métodos Java nativos do Android.
    * *Cenário:* Um atacante encontra um XSS na página carregada pela WebView -> O XSS chama a ponte Java -> O atacante ganha um RCE no dispositivo.

---

## O Elo Perdido: Intents e Deep Links

Como esses componentes conversam entre si? Eles usam o carteiro do sistema: os **Intents**. Um Intent é uma mensagem assíncrona que diz "Quero abrir a Câmera" ou "Quero iniciar o Serviço de Download".

### Intents

* **A Visão do Atacante:** Manipular Intents (**Intent Injection**) é a arte de fazer o sistema entregar uma mensagem maliciosa para um componente confiável, forçando-o a fazer algo que não deveria ou a vazar dados para um app atacante.

### Deep Links

Intents geralmente são internos, mas os **Deep Links**, ex: `nubank://transfer/` permitem que sites ou outros apps disparem Intents específicos dentro do seu aplicativo a partir de um link clicável.

* **Vetor de Ataque:** **Mobile CSRF**. Se o aplicativo não validar a origem do Deep Link, eu posso te mandar um link malicioso por SMS ou WhatsApp. Ao clicar, seu app bancário abre e preenche a tela de transferência automaticamente. Se o app tiver "Auto-Login" ou não exigir senha para essa ação específica, o ataque é executado instantaneamente.

## PendingIntents

Este é o vetor responsável por grandes CVEs recentes.
Um **PendingIntent** é um token que um app (A) cria e entrega para outro app (B), permitindo que B execute uma ação **com as permissões de A** no futuro.

* **O Ataque:** Se o App Vulnerável cria um PendingIntent vazio/genérico e o passa para um App Malicioso, o App Malicioso pode preencher esse Intent com o que quiser. Quando o App Vulnerável executar esse Intent, ele estará atacando a si mesmo com suas próprias permissões (ex: sobrescrevendo seus próprios arquivos ou garantindo permissões ao atacante).
* **Analogia:** É como assinar um cheque em branco e entregar para um estranho na rua.