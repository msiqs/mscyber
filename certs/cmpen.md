# Review: Certified Mobile Pentester (CMPen) - Android

**Data da Realização:** [Dez/2025]  
**Status:** [Aprovado]   
**Link Oficial:** [The SecOps Group - CNPen](https://pentestingexams.com/)

---

## 1. Executive Summary

A **CMPen-Android**, da The SecOps Group, é uma certificação focada na execução prática de testes de intrusão em aplicativos móveis. Diferente de exames teóricos, ela exige que você baixe um APK vulnerável, configure seu próprio ambiente de emulação e capture flags dentro de uma janela rígida de **4 horas**.

A proposta é validar a agilidade do pentester em identificar vulnerabilidades do OWASP Mobile Top 10 e realizar bypass de proteções em tempo real. Não é um exame sobre escrever relatórios bonitos, mas sobre provar que você consegue contornar controles de segurança (como SSL Pinning e Root Detection) e extrair dados sensíveis sob pressão.

Para profissionais de Offensive Security, a CMPen serve como um atestado de proficiência no uso de ferramentas de instrumentação dinâmica (**Frida/Objection**) em um ambiente sem restrições de tooling.

---

## 2. Pré-requisitos e Arsenal Técnico

O grande diferencial da CMPen é que **você é responsável pelo seu ambiente**. O exame não fornece uma VM no navegador. Você deve rodar o emulador na sua própria máquina e conectar via VPN apenas para validar as flags ou interagir com o servidor da aplicação. Ao contrário da EMAPT, por exemplo, que embora eu não tenha tirado ainda, já ouvi muitos dizerem que o ambiente virtual deles possui uma latência ridícula.

### 2.1. Setup Local
A preparação do laboratório é parte do teste. Se você não tiver isso pronto antes de clicar em Start, você perderá tempo precioso:
* **Emulação:** Android Studio AVD. Evite imagens com *Google Play Services* excessivos que poluem o tráfego do Burp.
* **Static Analysis:** *Jadx-GUI* leitura de código e *Apktool*.
* **Dynamic Instrumentation:** **Frida** é o coração deste exame. Você precisa ter seus scripts `.js` organizados para:
    * Bypass genérico de Root Detection.
    * Bypass de SSL Pinning (scripts do Codeshare ou customizados).
    * Hooking de classes para interceptar em tempo de execução.
* **Network:** Burp Suite configurado com o certificado CA instalado na raiz do sistema Android.

> **Ponto Crítico:** A liberdade de usar sua própria máquina é uma faca de dois gumes. Se o seu PC travar com a virtualização ou o ADB desconectar, o problema é seu. Garanta snapshots limpos do emulador antes da prova.

---

## 3. Estrutura do Exame

* **Duração:** 4 Horas.
* **Formato:** Capture The Flag (CTF)
* **Escopo:** Um único aplicativo `.apk` com múltiplas vulnerabilidades.
* **Passing Score:** 60% (75% para Mérito).

### 3.1. A Dinâmica
O fluxo é direto:
1.  Baixar o APK.
2.  Descompilar para achar hardcoded secrets e entender a lógica.
3.  Dynamic Analysis para interceptar requisições e manipular a lógica em tempo real.
4.  Explorar um pouco de API.
5.  Submeter a flag encontrada no portal do exame.

---

## 4. Análise Técnica dos Vetores

O exame é pragmático. Ele não pede que você reescreva o kernel do Android, mas exige domínio total da camada de aplicação e interação com o OS.

### 4.1. Análise Estática & Hardcoded Secrets
Muitas flags estão escondidas na má gestão de segredos.
* **Resources:** Análise minuciosa de `strings.xml` e arquivos de configuração dentro da pasta `/res`.
* **Código Java/Kotlin:** Identificação de chaves de API, credenciais de teste e URLs de *staging* esquecidas no código fonte.
* **Componentes Exportados:** Identificar Activities ou Content Providers que podem ser invocados via `adb shell am start` para pular autenticação.

### 4.2. Instrumentação Dinâmica
Aqui é onde a prova separa os curiosos dos profissionais.
* **SSL Pinning:** O aplicativo não vai confiar no seu certificado do Burp por padrão. Você deve ser capaz de derrubar essa proteção nos primeiros 15 minutos de prova.
* **Runtime Manipulation:** Algumas lógicas de validação ocorrem no lado do cliente. Você precisará usar o Frida para alterar o retorno de funções booleanas.
* **Storage Inseguro:** Buscar por dados sensíveis em `SharedPreferences` ou bancos SQLite (`/data/data/com.alvo...`) criados após a interação com o app.

---

## 5. Prós e Contras

### Pontos Fortes
* **Realismo de Ferramentas:** Você usa o mesmo ambiente que usaria em um pentest real de cliente, sem as limitações artificiais de uma VM web.
* **Custo-Benefício:** Geralmente mais barata que a subscrição da INE.
* **Foco em Frida:** A prova força você a perder o medo da linha de comando do Frida e entender como hooks funcionam de verdade.
* **Sem Burocracia:** Nada de relatórios. É Hack -> Flag -> Next.

### Pontos Fracos
* **Falta Profundidade:** Rodar um MOBSF e saber analisá-lo já mata quase metade da prova.

---

## 6. Conclusão Final

A **CMPen - Android** é uma certificação **honesta e tecnicamente ok** para o nível intermediário. Ela preenche a lacuna deixada pela mudança da eMAPT, oferecendo um desafio onde você tem controle total sobre as ferramentas de ataque.

É recomendada para quem quer validar a capacidade de **instrumentação dinâmica** e **bypass de proteções** em um cenário onde a agilidade conta mais do que a teoria.

**Nota Final:** ⭐⭐⭐⭐☆ (4/5)
*A melhor opção atual para quem prefere hackear na própria máquina ao invés de usar labs web.*

---