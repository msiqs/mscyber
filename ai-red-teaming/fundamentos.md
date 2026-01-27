# Fundamentos de IA: Tradução Completa do Material

## 1. Introdução ao Machine Learning

Na ciência da computação, os termos Inteligência Artificial (IA) e Machine Learning (ML) são frequentemente usados de forma intercambiável, gerando confusão. Embora intimamente relacionados, representam conceitos distintos.

### Inteligência Artificial (IA)
A IA é um campo amplo focado no desenvolvimento de sistemas inteligentes capazes de realizar tarefas que normalmente requerem inteligência humana. Essas tarefas incluem entender linguagem natural, reconhecer objetos, tomar decisões, resolver problemas e aprender com a experiência.
Algumas áreas-chave da IA incluem:
* **Processamento de Linguagem Natural (NLP):** Permitir que computadores entendam e gerem linguagem humana.
* **Visão Computacional:** Permitir que computadores "vejam" e interpretem imagens.
* **Robótica:** Robôs que realizam tarefas autonomamente.
* **Sistemas Especialistas:** Sistemas que imitam a tomada de decisão de especialistas humanos.

O objetivo principal é aumentar as capacidades humanas, não apenas substituí-las, auxiliando em análises complexas e produtividade. Exemplos práticos incluem diagnóstico de doenças, detecção de fraudes financeiras e mitigação de ameaças cibernéticas.

### Machine Learning (ML)
ML é um subcampo da IA focado em permitir que sistemas aprendam com dados e melhorem seu desempenho sem programação explícita. Algoritmos de ML usam estatística para identificar padrões e anomalias.

O ML é categorizado em três tipos principais:
1.  **Aprendizado Supervisionado:** O algoritmo aprende com dados rotulados (com resposta conhecida). Exemplos: Classificação de imagem, detecção de spam, prevenção de fraude.
2.  **Aprendizado Não Supervisionado:** O algoritmo aprende com dados não rotulados. Exemplos: Segmentação de clientes, detecção de anomalias, redução de dimensionalidade.
3.  **Aprendizado por Reforço (Reinforcement Learning):** Aprende por tentativa e erro através de recompensas e penalidades. Exemplos: Jogos, robótica, direção autônoma.

### Deep Learning (DL)
O Deep Learning é um subcampo do ML que utiliza redes neurais com múltiplas camadas para extrair características de dados complexos. É especialmente poderoso para dados não estruturados como imagens, áudio e texto.
Principais características:
* **Aprendizado Hierárquico:** Camadas inferiores detectam bordas/texturas; camadas superiores identificam formas/objetos complexos.
* **Escalabilidade:** Escala bem com grandes volumes de dados (Big Data).
* **Tipos de Redes:** CNNs (para imagens), RNNs (para dados sequenciais/texto) e Transformers (para NLP e atenção).

---

## 2. Reforço Matemático para IA

Este módulo aborda os conceitos matemáticos por trás dos algoritmos.

### Operações Algébricas e Notações
* **Notação de Subscrito ($x_t$):** Indica um estado específico em uma sequência ou tempo $t$. Ex: $x_t = q(x_t | x_{t-2})$.
* **Notação de Sobrescrito ($x^n$):** Denota expoentes. Ex: $x^2 = x * x$.
* **Símbolo de Somatório ($\Sigma$):** Indica a soma de uma sequência de termos. Ex: $\Sigma_{i=1}^{n} a_i$ representa a soma de $a_1$ até $a_n$.

### Normas Vetoriais
A norma mede o tamanho ou comprimento de um vetor:
* **Norma Euclidiana:** $||v|| = \sqrt{v_1^2 + v_2^2 + ... + v_n^2}$.
* **Norma L1 (Manhattan):** Soma dos valores absolutos.

### Logaritmos e Exponenciais
* **Logaritmo Base 2 ($log_2(x)$):** Usado em teoria da informação (entropia). Ex: $log_2(8) = 3$.
* **Logaritmo Natural ($ln(x)$):** Base $e$ (Euler). Usado em cálculo e probabilidade.
* **Função Exponencial ($e^x$):** Modela crescimento e distribuições de probabilidade.

### Operações com Matrizes e Vetores
* **Multiplicação Matriz-Vetor ($A*v$):** Transforma vetores. Fundamental para redes neurais.
* **Multiplicação Matriz-Matriz ($A*B$):** Usada em transformações lineares e operações entre camadas de Deep Learning.
* **Transposta ($A^T$):** Troca linhas por colunas.
* **Inversa ($A^{-1}$):** A matriz que, multiplicada por $A$, resulta na matriz identidade. Usada para resolver sistemas lineares.
* **Determinante ($det(A)$):** Valor escalar que determina se a matriz é invertível e está relacionado a volumes geométricos.
* **Traço ($tr(A)$):** Soma dos elementos da diagonal principal.

### Teoria dos Conjuntos
* **Cardinalidade ($|S|$):** Número de elementos em um conjunto.
* **União ($\cup$):** Elementos em A ou B.
* **Interseção ($\cap$):** Elementos em ambos A e B.

### Autovalores e Autovetores
* **Autovalor ($\lambda$):** Escalar que representa o fator de escala na transformação. $A*v = \lambda*v$.
* **Autovetor ($v$):** Vetor não nulo que não muda de direção (apenas escala) quando multiplicado pela matriz. Usado em PCA (Análise de Componentes Principais).

### Probabilidade e Estatística
* **Probabilidade Condicional ($P(x|y)$):** Probabilidade de $x$ dado $y$.
* **Esperança ($E[X]$):** Valor médio esperado de uma variável aleatória.
* **Variância ($Var(X)$):** Mede a dispersão dos dados em torno da média. $Var(X) = E[(X - E[X])^2]$.
* **Desvio Padrão ($\sigma(X)$):** Raiz quadrada da variância.
* **Covariância ($Cov(X,Y)$):** Mede como duas variáveis variam juntas.
* **Correlação ($\rho(X,Y)$):** Covariância normalizada (entre -1 e 1).

---

## 3. Algoritmos de Aprendizado Supervisionado

No aprendizado supervisionado, o algoritmo aprende uma função de mapeamento a partir de dados rotulados (exemplos com respostas corretas).

**Conceitos Centrais:**
* **Dados de Treino:** Conjunto de dados rotulados (features + labels).
* **Features (Características):** Propriedades mensuráveis dos dados (ex: tamanho da casa).
* **Labels (Rótulos):** A resposta correta ou variável alvo (ex: preço da casa).
* **Avaliação:** Métricas como Acurácia, Precisão, Recall e F1-Score.
* **Overfitting:** O modelo decora o treino e não generaliza. **Underfitting:** O modelo é simples demais.

### Regressão Linear
Algoritmo fundamental para predizer uma **variável contínua**. Assume uma relação linear entre as variáveis preditoras ($x$) e a variável alvo ($y$).
* **Equação:** $y = mx + c$ (Simples) ou $y = b_0 + b_1x_1 + ... + b_nx_n$ (Múltipla).
* **Método:** Usa Mínimos Quadrados Ordinários (OLS) para minimizar a soma dos erros quadráticos (a diferença entre o real e o predito).
* **Assunções:** Linearidade, Independência, Homocedasticidade (variância constante dos erros) e Normalidade dos erros.

### Regressão Logística
Apesar do nome, é usada para **classificação binária** (duas classes).
* **Função Sigmoide:** Mapeia qualquer valor de entrada para um probabilidade entre 0 e 1. Fórmula: $P(x) = 1 / (1 + e^{-z})$.
* **Fronteira de Decisão:** O limiar (threshold), geralmente 0.5, que define se a classe é 0 ou 1.
* **Assunções:** Resultado binário, relação linear das log-odds, e pouca multicolinearidade.

### Árvores de Decisão
Modelo intuitivo que cria regras de decisão simples baseadas nas features.
* **Estrutura:** Nó Raiz, Nós Internos (regras) e Nós Folha (resultado final).
* **Critérios de Divisão:**
    * **Impureza de Gini:** Probabilidade de classificar incorretamente um elemento. Quanto menor, mais puro.
    * **Entropia e Ganho de Informação:** Mede a desordem. O algoritmo busca o maior ganho de informação (maior redução da entropia).
* **Vantagens:** Não assume linearidade nem normalidade dos dados; robusto a outliers.

### Naive Bayes
Algoritmo probabilístico baseado no **Teorema de Bayes**.
* **Teorema de Bayes:** $P(A|B) = [P(B|A) * P(A)] / P(B)$. Atualiza a crença sobre um evento dado uma nova evidência.
* **"Naive" (Ingênuo):** Assume que as features são independentes entre si (ex: a presença de uma palavra não afeta a outra), o que simplifica o cálculo.
* **Tipos:** Gaussian (dados contínuos), Multinomial (contagem de palavras/texto), Bernoulli (binário).

### Support Vector Machines (SVM)
Busca encontrar o **hiperplano ideal** que maximiza a margem entre as classes.
* **Margem:** Distância entre o hiperplano e os pontos de dados mais próximos (chamados de vetores de suporte).
* **Kernel Trick:** Para dados não separáveis linearmente, projeta os dados em uma dimensão superior onde um plano linear pode separá-los. Kernels comuns: Polinomial, RBF (Radial Basis Function).
* **Função Objetivo:** Minimizar a magnitude do vetor de pesos $||w||^2$ mantendo a classificação correta.

---

## 4. Algoritmos de Aprendizado Não Supervisionado

Explora dados não rotulados para descobrir padrões ocultos, agrupamentos ou anomalias.

### Clustering (Agrupamento)
* **K-Means:** Particiona dados em $K$ grupos distintos.
    1.  Inicializa $K$ centróides aleatoriamente.
    2.  Atribui cada ponto ao centróide mais próximo (Distância Euclidiana).
    3.  Recalcula os centróides (média dos pontos).
    4.  Repete até estabilizar.
    * **Escolha do K:**
        * **Método do Cotovelo (Elbow):** Plota a soma dos erros quadráticos (WCSS). O ponto onde a curva "dobra" é o K ideal.
        * **Análise de Silhueta:** Mede o quão similar um ponto é ao seu cluster versus outros clusters (valor entre -1 e 1).

### Redução de Dimensionalidade (PCA)
O **Principal Component Analysis (PCA)** transforma dados de alta dimensão em uma representação menor, preservando a variância máxima.
* **Autovalores e Autovetores:** A equação $C * v = \lambda * v$ (onde C é a matriz de covariância) identifica os componentes principais.
* **Processo:** Padronizar dados -> Calcular matriz de covariância -> Calcular autovetores -> Selecionar os top $k$ autovetores -> Transformar os dados.

### Detecção de Anomalias
Identifica dados que desviam do padrão normal (outliers).
* **One-Class SVM:** Aprende uma fronteira que envolve os dados normais; tudo fora é anomalia.
* **Isolation Forest:** Isola anomalias particionando aleatoriamente os dados. Anomalias são isoladas mais rápido (caminhos mais curtos na árvore).
* **Local Outlier Factor (LOF):** Baseado em densidade. Compara a densidade local de um ponto com a de seus vizinhos.

---

## 5. Algoritmos de Aprendizado por Reforço (RL)

Um agente aprende a tomar decisões interagindo com um ambiente e recebendo feedback (recompensas/penalidades).

**Conceitos:**
* **Agente:** O tomador de decisão.
* **Ambiente:** O contexto onde o agente opera.
* **Estado ($S$):** Situação atual.
* **Ação ($A$):** Movimento do agente.
* **Recompensa ($R$):** Feedback imediato.
* **Política:** Estratégia que mapeia estados para ações.

### Q-Learning
Algoritmo *Model-Free* e *Off-Policy*. Aprende o **Q-Value** (recompensa futura esperada) para pares estado-ação.
* **Q-Table:** Tabela que armazena os valores Q.
* **Equação de Atualização (Bellman):**
    $Q(s,a) = Q(s,a) + \alpha * [r + \gamma * max(Q(s', a')) - Q(s,a)]$.
    Onde $\alpha$ é a taxa de aprendizado e $\gamma$ é o fator de desconto (importância do futuro).

### SARSA (State-Action-Reward-State-Action)
Algoritmo *On-Policy*. Atualiza os valores Q baseando-se na ação real que o agente tomará no próximo passo (seguindo sua política atual).
* **Diferença:** O Q-Learning assume que o agente tomará a *melhor* ação possível no futuro (max). O SARSA assume que o agente seguirá sua política atual (que pode incluir erros exploratórios), tornando-o mais conservador e seguro.

### Exploração vs. Explotação
* **Exploração:** Tentar novas ações para descobrir recompensas maiores.
* **Explotação:** Usar o conhecimento atual para maximizar a recompensa imediata.
* **Estratégia Epsilon-Greedy:** Com probabilidade $\epsilon$, escolhe uma ação aleatória (explora); com probabilidade $1-\epsilon$, escolhe a melhor ação conhecida (explota).

---

## 6. Deep Learning

Usa Redes Neurais Artificiais (ANNs) com múltiplas camadas para aprender padrões complexos.

### Componentes de uma Rede Neural
* **Neurônio (Perceptron):** Unidade básica. Calcula uma soma ponderada das entradas + viés ($b$) e aplica uma função de ativação.
* **Funções de Ativação:** Introduzem não-linearidade.
    * **ReLU:** Retorna 0 se negativo, valor real se positivo. Rápida e eficiente.
    * **Sigmoide/Tanh:** Comprimem valores para intervalos específicos (0 a 1 ou -1 a 1).
    * **Softmax:** Transforma saídas em distribuição de probabilidade (usada na última camada para classificação).

### Treinamento (Backpropagation)
1.  **Forward Pass:** Dados entram, rede faz a predição.
2.  **Cálculo do Erro (Loss):** Diferença entre predição e real.
3.  **Backward Pass (Retropropagação):** Calcula o gradiente do erro em relação aos pesos.
4.  **Otimização (Gradient Descent):** Atualiza os pesos na direção oposta ao gradiente para minimizar o erro.

### Redes Neurais Convolucionais (CNNs)
Projetadas para dados em grade (imagens).
* **Camadas Convolucionais:** Usam filtros (kernels) que deslizam sobre a imagem para extrair features (bordas, texturas). Gera mapas de características.
* **Pooling:** Reduz a dimensionalidade (ex: Max Pooling pega o maior valor de uma região), tornando a rede mais leve e robusta.
* **Arquitetura:** Camadas iniciais veem detalhes simples (linhas); camadas profundas veem objetos complexos (rostos, carros).

### Redes Neurais Recorrentes (RNNs)
Para dados sequenciais (texto, tempo). Possuem loops que permitem "memória" de estados passados.
* **Problema do Gradiente Evanescente:** Em sequências longas, o gradiente tende a zero durante a retropropagação, fazendo a rede "esquecer" o início da sequência.
* **Solução (LSTMs e GRUs):** Usam "portões" (gates) para controlar o fluxo de informação, decidindo o que lembrar ou esquecer, resolvendo o problema de dependência de longo prazo.

---

## 7. IA Generativa e Large Language Models (LLMs)

A IA Generativa cria novos dados (texto, imagem, código) em vez de apenas classificar dados existentes.

### Modelos de Linguagem (LLMs)
Baseados na arquitetura **Transformer**, processam texto em paralelo usando mecanismos de atenção.
* **Tokenização:** Quebra o texto em unidades menores (tokens).
* **Embeddings:** Representação numérica onde palavras com significados similares ficam próximas matematicamente.
* **Self-Attention:** Permite que o modelo entenda o contexto e a relação entre palavras distantes em uma frase (ex: saber a quem "ele" se refere num parágrafo anterior).

### Diffusion Models (Modelos de Difusão)
Usados para gerar imagens de alta qualidade (ex: criar uma imagem a partir de texto).
1.  **Processo Forward (Ruído):** Adiciona ruído gradualmente à imagem até virar aleatoriedade pura.
2.  **Processo Reverse (Denoising):** Uma rede neural aprende a remover o ruído passo a passo para reconstruir uma imagem nítida, condicionada por um texto (prompt).