# O Android: Um Linux Paranoico

Mobile hacking vem crescendo exponencialmente em demanda e complexidade. Pela abordagem ser diferente de Web ou Infra, muitos criam barreiras mentais. Sendo assim, criei esse conteúdo para desmistificar o android hacking.

Para atacar o Android com eficiência, a primeira chave mental que você precisa virar é entender que ele é, em sua essência, um **Linux Paranoico**.

Diferente de uma distribuição desktop onde o usuário tem ampla liberdade, o Android foi arquitetado sob o princípio do menor privilégio. Cada aplicativo é tratado como um inimigo potencial, rodando com seu próprio User ID e isolado em uma sandbox. O sistema não confia nos aplicativos, e os aplicativos não confiam uns nos outros. Nossa missão é encontrar onde essa "paranoia" falha.

## Arquitetura e Superfície de Ataque

Antes de sitiar o reino, precisamos mapear o terreno. O Android é estruturado em camadas. Para um pentester, cada camada exige ferramentas e mindsets específicos.

![Figura 1 - Ilustração da arquitetura](image.png)

## 1. System Apps

Esta é a elite do sistema. Habitam diretórios protegidos (`/system/app` ou `/system/priv-app`) e são Read-Only, exceto com Root.

* **O Alvo:** Apps de sistema frequentemente possuem permissões privilegiadas `signature` ou `system`.
* **Vetor de Ataque:** Se você encontrar uma vulnerabilidade em um app pré-instalado pelo fabricante, você pode conseguir execução de código com privilégios de sistema, o que é quase tão poderoso quanto o Root. É a porta de entrada para persistência de malware avançado que sobrevive ao "Factory Reset" em alguns casos.

## 2. Java API Framework

É a camada que expõe o hardware para os apps. Para Offensive Security, este é o palco da **Engenharia Reversa Dinâmica**.

É aqui que manipulamos a lógica de negócios. Ferramentas como **Frida** brilham nesta camada. As vulnerabilidades aqui geralmente são de **Lógica e IPC**, Intents mal configurados, Broadcasts perigosos e vazamento de dados via Logs.

## 3. Native C/C++ Libraries

Abaixo do conforto do Java, reside o "submundo" das bibliotecas nativas. O Android delega tarefas de alta performance (Renderização, Áudio, SSL, Processamento de Imagem) para código C/C++.

* **Vetor de Ataque:** **Memory Corruption**. Diferente do Java, aqui não há Garbage Collector. Erros manuais de gerenciamento de memória levam a Buffer Overflows, Use-After-Free e Integer Overflows. Um exploit aqui bypassa totalmente a Sandbox do Java.

## 4. Android Runtime (ART) 

Aqui reside uma diferença crucial para o pentester moderno.
Antigamente (Android 4.4-), usava-se a Dalvik (JIT). O Android moderno usa o **ART**, que opera num modelo híbrido complexo:

1.  **Instalação:** O código não é 100% compilado na hora, para economizar espaço.
2.  **Execução:** O código começa sendo **Interpretado**.
3.  **JIT (Just-In-Time):** Se um trecho de código é muito usado, ele é compilado para nativo em tempo real.
4.  **AOT (Ahead-Of-Time):** Quando o celular está carregando e ocioso, o daemon compila tudo para código de máquina nativo `.oat` / `.elf`.

**Por que isso importa para o Hacking?**
Isso afeta diretamente o **Hooking**. Ferramentas como o Frida precisam lidar com métodos que podem estar em estados diferentes (interpretados vs. nativos). Às vezes, um hook falha porque o ART "inlinou" o método. Entender o ciclo de vida do Zygote é vital para criar hooks persistentes.

## 5. Hardware Abstraction Layer (HAL)

A HAL é onde o Android genérico do Google encontra o hardware proprietário da Samsung, Xiaomi, Motorola.

* **Vetor de Ataque:** **Drivers Proprietários**. O código do Google é auditado. O código do driver da câmera ou do sensor de digital feito às pressas pelo fabricante do chip... nem sempre. Muitas falhas de escalação de privilégio local (LPE) nascem aqui.

## 6. Linux Kernel

O alicerce. Baseado no Linux, mas altamente modificado (wakelocks, binder, ashmem).

* **Binder (IPC):** O "carteiro" do Android. Quase toda comunicação entre processos passa por aqui. Explorar o driver do Binder permite interceptar mensagens globais do sistema.
* **Rooting:** Ganhar acesso ao Kernel é o objetivo final. Com Root, as Sandboxes desaparecem, o SELinux pode ser desligado e você se torna o "Deus" do dispositivo.