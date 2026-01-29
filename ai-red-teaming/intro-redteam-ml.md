# Introdução ao Red Teaming em Sistemas Baseados em Machine Learning

Para avaliar a segurança de sistemas baseados em Machine Learning (ML) com competência, é essencial possuir um entendimento profundo dos componentes subjacentes e dos algoritmos que regem esses sistemas. Devido à significativa complexidade inerente a estas tecnologias, existe uma vasta superfície de ataque onde problemas de segurança podem surgir. Antes de discutirmos e demonstrarmos técnicas práticas que podemos alavancar ao avaliar a segurança de sistemas baseados em ML, é crucial estabelecer uma fundação teórica adequada. Estes sistemas englobam diferentes componentes interconectados e, ao longo deste documento, exploraremos uma visão abrangente dos riscos de segurança e vetores de ataque em cada um deles.

## O que é Red Teaming?

Tradicionalmente, ao discutir avaliações de segurança de sistemas de Tecnologia da Informação, o tipo mais comum de avaliação encontrado no mercado é o Teste de Intrusão, ou Penetration Test. Este tipo de avaliação é tipicamente um exercício focado e limitado pelo tempo, visando descobrir e explorar vulnerabilidades em sistemas específicos, aplicações ou ambientes de rede. Profissionais de Pentest seguem um processo estruturado, frequentemente utilizando ferramentas automatizadas combinadas com técnicas de teste manual para identificar fraquezas de segurança dentro de um escopo pré-definido. O objetivo principal de um teste de intrusão é determinar se vulnerabilidades existem, se elas podem ser exploradas e qual a extensão do dano potencial. Geralmente, esses testes são realizados em segmentos de rede isolados ou instâncias de aplicação dedicadas para evitar interferência com usuários reais em produção.

No entanto, é comum distinguir dois tipos adicionais de avaliação de segurança: Red Team Assessments e Vulnerability Assessments.

### Vulnerability Assessments (Avaliação de Vulnerabilidades)
As avaliações de vulnerabilidade são processos geralmente mais automatizados que focam na identificação, catalogação e priorização de vulnerabilidades conhecidas dentro da infraestrutura de uma organização. Diferente do Pentest, estas avaliações tipicamente não envolvem a exploração ativa das falhas, mas focam primordialmente na identificação das mesmas. Elas fornecem uma varredura abrangente de sistemas, aplicações e redes para identificar lacunas de segurança potenciais que poderiam ser exploradas. Estes resultados são frequentemente fruto de varreduras automatizadas utilizando scanners de vulnerabilidade de mercado, como o Nessus ou OpenVAS.

### Red Team Assessment
Este é o foco principal deste estudo. O termo descreve uma simulação adversarial avançada onde especialistas de segurança, conhecidos como o Red Team, imitam as Táticas, Técnicas e Procedimentos (TTPs) de atacantes do mundo real para testar as defesas de uma organização. O objetivo do Red Team é explorar vulnerabilidades técnicas e desafiar cada aspecto da segurança da organização, incluindo o fator humano e os processos internos, empregando táticas como engenharia social, phishing e até intrusões físicas. As avaliações de Red Team focam na furtividade e na persistência, trabalhando ativamente para evitar a detecção pelo time de defesa, o Blue Team, enquanto buscam meios de atingir objetivos estratégicos específicos, como acessar bancos de dados sensíveis ou comprometer sistemas críticos. Este exercício é extenso e muitas vezes abrange períodos de semanas a meses, fornecendo uma análise profunda da resiliência geral de uma organização contra ameaças sofisticadas.

## Red Teaming em Sistemas Baseados em ML

Diferente dos sistemas de TI tradicionais, os sistemas baseados em Machine Learning enfrentam vulnerabilidades únicas porque dependem de grandes conjuntos de dados, inferência estatística probabilística e arquiteturas de modelo complexas (muitas vezes "caixas-pretas"). Consequentemente, avaliações de Red Team são frequentemente a abordagem recomendada ao avaliar a segurança de sistemas baseados em ML, visto que muitas técnicas de ataque avançadas requerem mais tempo de análise e execução do que um teste de intrusão típico permitiria.

Além disso, sistemas baseados em ML são compostos por vários componentes que interagem entre si. Frequentemente, as vulnerabilidades de segurança surgem exatamente nestes pontos de interação. Portanto, incluir todos estes componentes na avaliação de segurança é benéfico. Determinar o escopo de um teste de intrusão para um sistema baseados em ML pode ser difícil e pode inadvertidamente excluir componentes específicos ou pontos de interação, tornando certas vulnerabilidades de segurança impossíveis de serem reveladas.

---

## Atacando Sistemas de ML (OWASP Top 10 ML)

Da mesma forma que existe para Aplicações Web, APIs e Aplicações Móveis, a OWASP publicou uma lista dos 10 principais riscos de segurança referentes à implantação e gerenciamento de sistemas baseados em ML. Discutiremos detalhadamente estes dez riscos para obter uma visão geral dos problemas de segurança resultantes destes sistemas.

### Visão Geral dos Riscos

1.  **ML01 - Input Manipulation Attack:** Atacantes modificam os dados de entrada para causar saídas incorretas ou maliciosas do modelo.
2.  **ML02 - Data Poisoning Attack:** Atacantes injetam dados maliciosos ou enganosos nos dados de treinamento, comprometendo o desempenho do modelo ou criando backdoors.
3.  **ML03 - Model Inversion Attack:** Atacantes treinam um modelo separado para reconstruir as entradas a partir das saídas do modelo alvo, potencialmente revelando informações sensíveis usadas no treino.
4.  **ML04 - Membership Inference Attack:** Atacantes analisam o comportamento do modelo para determinar se um dado específico foi incluído no conjunto de dados de treinamento do modelo, o que constitui uma violação de privacidade.
5.  **ML05 - Model Theft:** Atacantes treinam um modelo separado a partir das interações com o modelo original, roubando assim a propriedade intelectual.
6.  **ML06 - AI Supply Chain Attacks:** Atacantes exploram vulnerabilidades em qualquer parte da cadeia de suprimentos de ML (bibliotecas, dados de terceiros, modelos pré-treinados).
7.  **ML07 - Transfer Learning Attack:** Atacantes manipulam o modelo base que é subsequentemente refinado (fine-tuned) por terceiros. Isso pode levar a modelos enviesados ou com backdoors herdados.
8.  **ML08 - Model Skewing:** Atacantes enviesam o comportamento do modelo para propósitos maliciosos, por exemplo, manipulando o conjunto de dados de treinamento para favorecer uma classe específica.
9.  **ML09 - Output Integrity Attack:** Atacantes manipulam a saída de um modelo antes que ela seja processada pelo sistema subsequente, fazendo parecer que o modelo produziu um resultado diferente.
10. **ML10 - Model Poisoning:** Atacantes manipulam diretamente os pesos e parâmetros do modelo, comprometendo seu desempenho ou criando backdoors.

### Detalhamento: Input Manipulation Attack (ML01)
Como o nome sugere, ataques de manipulação de entrada compreendem qualquer tipo de ataque contra um modelo de ML que resulta da manipulação dos dados inseridos pelo usuário. Tipicamente, o resultado destes ataques é um comportamento inesperado do modelo de ML que se desvia do comportamento pretendido. O impacto depende altamente do cenário concreto e das circunstâncias em que o modelo é usado, podendo variar de danos financeiros e reputacionais a consequências legais ou perda de dados.

Muitos vetores de ataque de manipulação de entrada no mundo real aplicam pequenas perturbações a dados de entrada benignos. Estas perturbações são calculadas para serem tão pequenas que a entrada ainda parece benigna ao olho humano, mas resultam em comportamento inesperado pelo modelo.

* **Exemplo Prático:** Considere um carro autônomo que usa um sistema baseado em ML para classificação de imagens de sinais de trânsito para detectar o limite de velocidade atual ou sinais de pare. Em um ataque de manipulação de entrada, um atacante poderia adicionar pequenas perturbações, como pedaços de sujeira colocados estrategicamente, pequenos adesivos ou graffiti nos sinais de trânsito. Embora estas perturbações pareçam inofensivas para o olho humano, elas poderiam resultar na classificação incorreta do sinal pelo sistema (ex: interpretar um PARE como SIGA). Isso pode ter consequências fatais para os passageiros do veículo.

### Detalhamento: Data Poisoning Attack (ML02)
Ataques de envenenamento de dados em sistemas baseados em ML envolvem a injeção de dados maliciosos ou enganosos no conjunto de dados de treinamento para comprometer a precisão, desempenho ou comportamento do modelo. A qualidade de qualquer modelo de ML é altamente dependente da qualidade dos dados de treinamento. Como tal, estes ataques podem fazer com que um modelo faça previsões incorretas, classifique incorretamente certas entradas ou se comporte de forma imprevisível em cenários específicos. Modelos de ML frequentemente dependem de coleta de dados automatizada em larga escala de várias fontes, tornando-os mais suscetíveis a tal adulteração, especialmente quando as fontes não são verificadas ou são coletadas de domínios públicos.



* **Exemplo Prático:** Assuma que um adversário é capaz de injetar dados maliciosos no conjunto de dados de treinamento para um modelo usado em software antivírus para decidir se um determinado binário é malware. O adversário pode manipular os dados de treinamento para efetivamente estabelecer um **backdoor** que permite criar malwares personalizados que o modelo classifica sistematicamente como binários benignos, ignorando a detecção.

### Detalhamento: Model Inversion Attack (ML03)
Em ataques de inversão de modelo, um adversário treina um modelo de ML separado baseado na saída do modelo alvo para reconstruir informações sobre as entradas originais do modelo alvo. Como o modelo treinado pelo adversário opera na saída do modelo alvo e reconstrói informações sobre as entradas, ele inverte a funcionalidade do modelo alvo.

Estes ataques são particularmente impactantes se os dados de entrada contiverem informações sensíveis, por exemplo, modelos processando dados médicos, como classificadores usados na detecção de câncer. Se um modelo inverso pode reconstruir informações sobre os dados médicos de um paciente com base na saída do classificador, informações sensíveis correm o risco de serem vazadas para o adversário. Além disso, ataques de inversão de modelo são mais desafiadores de executar se o modelo alvo fornecer menos informações na saída. Por exemplo, treinar com sucesso um modelo inverso torna-se muito mais desafiador se um modelo de classificação apenas fornece a classe alvo final em vez da probabilidade de cada classe possível.

### Detalhamento: Membership Inference Attack (ML04)
Ataques de inferência de pertencimento buscam determinar se uma amostra de dados específica foi incluída no conjunto de dados de treinamento original do modelo. Ao analisar cuidadosamente as respostas do modelo a diferentes entradas, um atacante pode inferir quais pontos de dados o modelo "lembra" do processo de treinamento. Se um modelo é treinado em dados sensíveis, como informações médicas ou financeiras, isso pode apresentar sérios problemas de privacidade. Este ataque é especialmente preocupante em modelos acessíveis publicamente ou compartilhados, como aqueles em ambientes baseados em nuvem ou Machine-Learning-as-a-Service (MLaaS). O sucesso dos ataques de inferência de pertencimento frequentemente depende das diferenças no comportamento do modelo ao lidar com dados de treinamento versus dados de não-treinamento (generalização), pois modelos tipicamente exibem maior confiança ou menor erro de previsão em amostras que eles já viram antes (overfitting).

### Detalhamento: Model Theft (ML05)
O roubo de modelo, ou ataques de extração de modelo, visam duplicar ou aproximar a funcionalidade de um modelo alvo sem acesso direto à sua arquitetura subjacente ou parâmetros. Nestes ataques, um adversário interage com a API de um modelo de ML e o consulta sistematicamente para reunir dados suficientes sobre seu comportamento de tomada de decisão para duplicar o modelo. Ao observar saídas suficientes para várias entradas estratégicas, atacantes podem treinar seu próprio modelo réplica com um desempenho similar.

O roubo de modelo ameaça a propriedade intelectual de organizações que investem em modelos de ML proprietários, potencialmente resultando em danos financeiros ou reputacionais. Além disso, o roubo de modelo pode expor insights sensíveis incorporados dentro do modelo, como padrões aprendidos de dados de treinamento sensíveis.

---

## Estudo de Caso Prático: Manipulando o Modelo

Agora que exploramos vulnerabilidades de segurança comuns que surgem da implementação inadequada de sistemas baseados em ML, vamos analisar um exemplo prático. Exploraremos como um modelo de ML reage a mudanças nos dados de entrada e dados de treinamento para entender melhor como vulnerabilidades relacionadas à manipulação de dados surgem. Isso inclui ataques de manipulação de entrada (ML01) e ataques de envenenamento de dados (ML02).

Usaremos um classificador de spam baseado em **Naive Bayes** como linha de base.

### 1. Manipulando a Entrada (Input Manipulation/Evasion)

O código base treina um classificador e avalia sua precisão. Suponha que, ao rodar o classificador, ele forneça uma precisão sólida de 97.2%.

Para entender como o modelo reage a certas palavras na entrada, analisamos a inferência em uma única mensagem. A função de classificação retorna as probabilidades de saída do classificador em vez de apenas a classe prevista. Estamos usando um classificador de spam que classifica em duas classes: `ham` (classe 0, benigno) e `spam` (classe 1, malicioso).

* **Entrada Benigna:** `message = "Hello World! How are you doing?"`
    * **Resultado:** Classe `Ham`. Probabilidade Ham: 98.93%, Probabilidade Spam: 1.07%.
    * **Análise:** O modelo está muito confiante de que a mensagem é benigna.

* **Entrada Maliciosa:** `message = "Congratulations! You won a prize. Click here to claim: https://bit.ly/3YCN7PF"`
    * **Resultado:** Classe `Spam`. Probabilidade Ham: 0.0%, Probabilidade Spam: 100.0%.
    * **Análise:** O modelo detectou corretamente como spam.

Em um ataque de manipulação de entrada, nosso objetivo como atacantes é fornecer uma entrada ao modelo que resulte em classificação incorreta. Neste caso, tentaremos enganar o modelo para classificar uma mensagem de spam como ham.

#### Técnica A: Rephrasing (Reescrita)
Frequentemente, estamos interessados apenas em fazer a vítima clicar no link fornecido. Para evitar ser sinalizado por classificadores de spam, devemos considerar cuidadosamente as palavras que escolhemos. O classificador detecta facilmente a mensagem acima como spam por causa de palavras como "Congratulations" e "Prize".

Podemos testar como o modelo reage a partes isoladas.
* Apenas a palavra `Congratulations!` -> Probabilidade de Spam: 64.97%.
* A frase `You won a prize.` -> Probabilidade de Spam: 99.73%.

Com este conhecimento, podemos tentar palavras e frases diferentes que tenham uma baixa probabilidade de serem sinalizadas como spam. Se mudarmos a mensagem de entrada para um pretexto de engenharia social diferente, como: `"Your account has been blocked. You can unlock your account in the next 24h: LINK"`, a entrada poderá ser classificada, por pouco, como `Ham`, pois essas palavras são estatisticamente menos associadas a spam naquele modelo específico.

#### Técnica B: Overpowering (Sobreposição)
Outra técnica é sobrecarregar a mensagem de spam com palavras benignas para empurrar a decisão do classificador em direção a uma classe particular. Podemos conseguir isso simplesmente anexando palavras à mensagem de spam original até que o conteúdo "ham" sobreponha o conteúdo "spam" da mensagem.

Quando o classificador processa muitos indicadores de `ham`, ele acha esmagadoramente mais provável que a mensagem seja `ham`, mesmo que o conteúdo original de spam ainda esteja presente. Lembre-se que o algoritmo Naive Bayes assume que cada palavra contribui independentemente para a probabilidade final.

* **Ataque:** Anexamos o primeiro parágrafo de um texto clássico (como *Lorem Ipsum* ou um trecho de livro) ao final do link malicioso.
* **Resultado:** A probabilidade de `Ham` sobe para 57.39%, classificando a mensagem como benigna, permitindo que ela chegue à caixa de entrada da vítima. Esta técnica funciona particularmente bem em e-mails HTML onde podemos esconder as palavras anexadas em comentários HTML invisíveis ao usuário, mas visíveis ao classificador.

### 2. Manipulando os Dados de Treinamento (Data Poisoning)

Após explorar como a manipulação dos dados de entrada afeta a saída do modelo, vamos passar para os dados de treinamento. Para demonstrar isso, imagine criar um conjunto de dados de treinamento separado, significativamente reduzido, para que nossas manipulações tenham um efeito maior.

Se injetarmos entradas de dados falsas no conjunto (envenenamento), podemos alterar o comportamento do classificador.

* **Cenário:** O modelo original classifica a mensagem `"Hello World! How are you doing?"` como `Ham` com 98.7% de confiança.
* **Ataque:** Injetamos itens de dados adicionais no conjunto de treinamento CSV. Adicionamos itens rotulados como `spam` que contêm as frases da nossa mensagem benigna.
    * Injeção no CSV: `spam,"Hello World"`
    * Injeção no CSV: `spam,"How are you doing?"`
* **Resultado:** Após retreinar o modelo com o dado envenenado, a mesma mensagem `"Hello World! How are you doing?"` agora é classificada como `Spam` com 79.66% de probabilidade.

Forçamos o classificador a classificar incorretamente uma mensagem de entrada específica manipulando o conjunto de dados de treinamento. Conseguimos isso sem um efeito adverso substancial na precisão geral do modelo (que caiu apenas 0.4%), o que torna os ataques de envenenamento de dados poderosos e difíceis de detectar.

---

## Atacando Geração de Texto (OWASP Top 10 LLM)

Para começar a explorar vulnerabilidades de segurança que podem surgir ao usar sistemas baseados em IA generativa, vamos discutir vulnerabilidades específicas para geração de texto. O modelo de escolha para geração de texto são os Grandes Modelos de Linguagem (LLMs). A OWASP publicou uma lista Top 10 específica para aplicações de LLM.

### Destaques dos Riscos para LLMs

* **LLM01 - Prompt Injection:** Atacantes manipulam a entrada do LLM direta ou indiretamente para causar comportamento malicioso ou ilegal. Isso inclui exemplos aparentemente benignos, como enganar um chatbot de suporte técnico para fornecer receitas de culinária (Jailbreak), mas também pode levar a LLMs gerando informações deliberadamente falsas, discurso de ódio ou revelando instruções ocultas.
    
* **LLM02 - Insecure Output Handling:** O texto gerado por LLM deve ser tratado da mesma forma que a entrada de usuário não confiável. Se as aplicações web não validarem ou sanitizarem a saída do LLM adequadamente, vulnerabilidades comuns da web como Cross-Site Scripting (XSS), Injeção de SQL ou injeção de código podem surgir no backend que consome essa saída. Por exemplo, se um LLM gera uma query SQL baseada em texto e a aplicação executa essa query diretamente, um `DROP TABLE` gerado pelo modelo seria catastrófico.
* **LLM03 - Training Data Poisoning:** Similar ao ML02, mas específico para LLMs. A manipulação de dados de treino introduz vieses ou backdoors. Se um LLM é treinado em dados publicamente disponíveis, sanitizar os dados de treinamento é essencial.
* **LLM04 - Model Denial of Service:** Atacantes alimentam o LLM com entradas que resultam em alto consumo de recursos (janela de contexto expandida, loops de raciocínio), potencialmente causando interrupções no serviço.
* **LLM05 - Supply Chain Vulnerabilities:** Vulnerabilidades em plugins, modelos base ou datasets de terceiros.
* **LLM06 - Sensitive Information Disclosure:** Atacantes enganam o LLM para revelar informações sensíveis na resposta, seja através de engenharia social no prompt ou explorando a memorização de dados de treino (PII).
* **LLM07 - Insecure Plugin Design:** Atacantes exploram vulnerabilidades em plugins que conectam o LLM a sistemas externos (ex: um plugin que permite enviar emails sem confirmação do usuário).
* **LLM08 - Excessive Agency:** Ocorre quando é dada ao LLM mais autonomia do que o necessário. Similar ao princípio do menor privilégio, é vital restringir as capacidades de um LLM. Se um LLM pode interagir com um banco de dados, ele não deve ter permissões de `DELETE` ou `DROP` se sua função é apenas leitura.
* **LLM09 - Overreliance:** Uma organização confia excessivamente na saída de um LLM para decisões de negócios críticas, sem supervisão humana ("human in the loop"), levando a problemas de segurança devido a "alucinações" ou códigos inseguros gerados.
* **LLM10 - Model Theft:** Acesso não autorizado aos pesos e parâmetros do LLM proprietário.

---

## Framework de IA Segura do Google (SAIF)

Um framework adicional cobrindo riscos de segurança em aplicações de IA é o Secure AI Framework (SAIF) do Google. Ele fornece princípios acionáveis para o desenvolvimento seguro de todo o pipeline de IA - da coleta de dados à implantação do modelo. Enquanto a OWASP fornece uma lista técnica de vulnerabilidades, o SAIF oferece uma perspectiva mais ampla e holística.



### Áreas e Componentes do SAIF
No SAIF, existem quatro áreas diferentes de desenvolvimento de IA segura, que usaremos para categorizar a superfície de ataque:

1.  **Data:** Esta área consiste em todos os componentes relacionados a dados, como fontes de dados, filtragem, processamento e dados de treinamento.
2.  **Infrastructure:** Relaciona-se ao hardware onde a aplicação é hospedada, armazenamento de dados e plataformas de desenvolvimento. Inclui frameworks, código, processos de treino/tuning e o "Model Serving".
3.  **Model:** A área central. Compreende o próprio artefato do modelo, o manuseio de entrada e o manuseio de saída.
4.  **Application:** Relaciona-se à interação com a aplicação de IA, ou seja, consiste nas aplicações que interagem com a implantação da IA e potenciais Agentes ou Plugins.

O SAIF fornece um **Mapa de Riscos** que conecta estes componentes a controles de mitigação. Por exemplo, para o risco de "Prompt Injection", o controle sugerido é "Validação e Sanitização de Entrada e Saída", e a responsabilidade é compartilhada entre os Criadores do Modelo e os Consumidores do Modelo.

---

## Red Teaming em IA Generativa: Os 4 Componentes de Ataque

Ao realizar um Red Team em sistemas de IA, devemos adotar Táticas, Técnicas e Procedimentos (TTPs) adaptados. Cada um dos quatro componentes discutidos acima possui desafios únicos.

### 1. Atacando Componentes de Modelo
O componente de modelo consiste em tudo diretamente relacionado ao modelo de ML em si (pesos, vieses, processo de treino).
* **Natureza Black-box:** Entender por que um modelo reage de certa maneira é difícil. Testes de segurança devem adotar um estilo "caixa-preta", enviando muitos inputs e analisando os outputs para mapear fronteiras de decisão.
* **Riscos Principais:** Model Poisoning, Evasion Attacks (Jailbreaks para ignorar diretrizes de segurança), Model Extraction (roubo de IP).
* **TTPs:** Consultas adaptativas, fuzzing de prompts, análise estatística de respostas.

### 2. Atacando Componentes de Dados
O componente de dados é crítico pois a qualidade do modelo depende dele.
* **Riscos Principais:** Data Poisoning, Data Leaks.
* **TTPs:** Comprometimento da cadeia de suprimentos de dados (datasets open-source envenenados), exploração de armazenamento em nuvem mal configurado contendo datasets, Insider Threat (funcionários exfiltrando dados).

### 3. Atacando Componentes de Aplicação
Este componente refere-se à aplicação tradicional onde a IA está integrada (Web App, API).
* **Riscos Principais:** Injection (SQL Injection ou Command Injection causados por saída de IA não sanitizada), Autenticação Quebrada, XSS.
* **TTPs:** Payloads de XSS gerados via Prompt Injection e renderizados no navegador da vítima, ataques de engenharia social aprimorados por IA, exploração de vulnerabilidades web clássicas na interface do chatbot.

### 4. Atacando Componentes de Sistema
Inclui hardware, sistema operacional, configuração de nuvem e deployment.
* **Riscos Principais:** Misconfigurations (portas de administração expostas, credenciais padrão em servidores Jupyter/MLflow), DoS (exaustão de recursos computacionais/GPU).
* **TTPs:** Escaneamento de vulnerabilidades em infraestrutura, Password Spraying, ataques volumétricos ou de complexidade algorítmica para negar serviço.

---