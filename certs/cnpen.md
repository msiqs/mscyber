# Review: Certified Network Pentester (CNPen) - The SecOps Group

**Data da Realização:** [Mar/2025]  
**Status:** [Aprovado]  
**Versão do Exame:** [Atual]  
**Link Oficial:** [The SecOps Group - CNPen](https://pentestingexams.com/)

---

## 1. Executive Summary

A **Certified Network Pentester (CNPen)**, oferecida pelo *The SecOps Group*, posiciona-se como uma certificação de nível intermediário focada exclusivamente na execução prática de testes de intrusão em redes. Diferente de certificações teóricas ou daquelas que exigem relatórios extensos, como a PNPT ou OSCP, a CNPen adota uma abordagem pragmática e ágil: um exame curto, intenso e orientado a desafios em um ambiente corporativo simulado.

Este review disseca a estrutura do exame, a profundidade técnica exigida e avalia se o investimento faz sentido para profissionais que buscam validar competências em *Network Hacking* sem necessariamente passar pelo processo de documentação formal exigido em exames de 24h ou 48h.

Minha análise é baseada na perspectiva de um profissional de Offensive Security focado em eficiência técnica e realismo de cenários. Sendo assim, aqui vai o primeiro disclaimer: 

Embora apresentada como mid-level, ou nível intermediário, quando tratamos de um cenário de pentest real, a CNPen não apresenta um sistema estruturado e maduro, dessa forma, acaba distoando um pouco nesse aspecto. Mas, acredito que isso possa ser porque de fato não estamos falando de uma certificação de nível expert ou avançado.

---

## 2. Pré-requisitos e Preparação

Antes de iniciar o exame, é crucial entender que a CNPen não oferece um curso oficial obrigatório que segure sua mão. A preparação exige autonomia. O syllabus cobre vetores de ataque essenciais que qualquer Pentester Júnior deve dominar.

### 2.1. Conhecimentos Exigidos (Syllabus Breakdown)
A grade curricular é focada na "carne" do pentest de rede, ignorando teorias de gestão ou compliance. Os domínios principais que precisei dominar incluíram:

* **Network Mapping & Identification:** Uso avançado de Nmap e identificação precisa de serviços.
* **Vulnerability Assessment:** Capacidade de correlacionar versões de serviços com CVEs públicas e misconfigs.
* **Exploitation:** Uso de public exploits, Metasploit Framework e Password Attacks.
* **Post-Exploitation & Privilege Escalation:** Enumeração interna em Linux e Windows para escalar privilégios verticalmente.

### 2.2. Meu Laboratório de Estudos
Para me preparar, utilizei uma abordagem prática focada nas seguintes plataformas, visto que o exame é 100% hands-on:

* **TryHackMe/HackTheBox:** Foco em máquinas "Medium" que exigem enumeração meticulosa.
* **PortSwigger Academy:** Para os componentes de Web Application que podem aparecer como vetor de entrada na rede.
* **Ferramentas Chave:** Aprofundei o domínio em Burp Suite, *Nmap* e scripts personalizados em *Python/Bash*.

> **Nota Crítica:** Se você depende exclusivamente de ferramentas automatizadas, scanners de vulnerabilidade "point-and-click" como Nessus, você falhará. O exame exige validação manual e exploração customizada.

---

## 3. O Exame: Estrutura e Ambiente

O exame da CNPen rompe com o modelo tradicional de "maratona".

* **Duração:** 4 Horas.
* **Questões:** 15 Questões Práticas.
* **Formato:** Capture The Flag / Flag Submission.
* **Passing Score:** 60% para aprovação (75% para mérito).

### 3.1. Conectividade e VPN
O acesso ao ambiente é feito via OpenVPN. Na minha experiência em específico eu nunca tive problemas com VPNs em certificações da SecOps.

### 3.2. A Dinâmica de "Speed Run"
Quatro horas é um tempo extremamente agressivo para um pentest de rede, mesmo que em escopo reduzido. Isso muda a psicologia da prova: não há tempo para rabbit holes. A metodologia deve ser afiada:
1.  **Scan Rápido:** Identificação imediata de low-hanging fruits.
2.  **Exploração Simultânea:** Enquanto uma wordlist roda no fundo, você deve estar enumerando outro serviço manualmente.
3.  **Gerenciamento de Estresse:** O relógio é o maior adversário.

---

## 4. Análise Técnica dos Cenários

O ambiente do exame simula uma sub-rede corporativa com múltiplos hosts. A topologia não é plana; exige entendimento de como navegar entre serviços.

### 4.1. Recon
A fase de enumeração não é trivial. Os serviços não estão simplesmente "lá" esperando serem pegos.
* **Service Discovery:** Foi necessário ir além do `-sV` padrão do Nmap. A análise de banners e a interação manual com portas foram essenciais para identificar serviços modificados ou rodando em portas não padrão.
* **OSINT:** O exame flerta com elementos de inteligência de fontes abertas, exigindo que você busque informações fora do ambiente confinado para conseguir acesso inicial.

### 4.2. Exploitation
A dificuldade técnica das falhas situa-se no nível **Intermediário**.
* Não encontrei 0-days ou explorações de binários complexos Buffer Overflow avançado, mas sim uma forte ênfase em **misconfigurations** e **vulnerabilidades conhecidas** que exigem modificação de exploit code.
* **Web Vectors:** Existe um momento que exige-se exploração de aplicações web (LFI, SQLi ou RCE simples) para obter a shell inicial.

### 4.3. PrivEsc
Esta foi a parte mais técnica e satisfatória.
* **Linux:** Enumeração de permissões SUID, *cron jobs* mal configurados e kernel exploits antigos.
* **Windows:** O foco recaiu sobre serviços rodando com privilégios de SYSTEM, *unquoted service paths* ou credenciais armazenadas em texto claro.
* A "pegadinha" aqui é saber diferenciar o que é um vetor real de escalação do que é apenas "barulho" do sistema operacional.

---

## 5. Prós e Contras

Uma visão analítica e fria sobre o produto oferecido.

### Pontos Fortes
* **Realismo de Tempo:** Simula a pressão de um *engagement* real onde o cliente exige resultados rápidos ou uma triagem de emergência.
* **Custo-Benefício:** Em comparação com certificações da OffSec, o preço da CNPen é uma fração, entregando um ROI ok para validação técnica.
* **Sem Report:** Para profissionais que odeiam a burocracia de relatórios de 50 páginas apenas para passar em uma prova, o modelo de *flag submission* é libertador e foca puramente na skill técnica.
* **Ambiente Estável:** A infraestrutura suportou bem as ferramentas de brute-force sem travar os hosts, algo comum em labs baratos.

### Pontos Fracos
* **Falta de Feedback Detalhado:** Como o sistema é baseado em flags, você não sabe por que errou uma questão específica, apenas que a flag estava incorreta.
* **Janela de Tempo Curta:** 4 horas podem penalizar bons profissionais que têm uma metodologia mais lenta e metódica, favorecendo quem tem "memória muscular" de CTF.
* **Reconhecimento de Mercado:** Ainda não possui o peso de RH de uma OSCP. É uma certificação para provar competência técnica para si mesmo ou para times técnicos, não necessariamente para passar em filtros automáticos de recrutadores leigos.

---

### Conclusão Final
A **Certified Network Pentester** é uma certificação **honesta, direta e tecnicamente válida**. Ela cumpre o que promete: testar sua capacidade de enumerar e comprometer redes em um curto espaço de tempo.

Recomendo para:
1.  Estudantes que buscam uma validação prática antes de investir milhares de reais em certificações de elite.
2.  Profissionais que precisam afiar sua metodologia de *Time Management*.
3.  Pentesters que buscam diversificar o portfólio com certificações focadas em *hands-on* puro.

**Nota Final:** ⭐⭐⭐⭐☆ (4/5)
Uma excelente prova de fogo para a metodologia de enumeração rápida.

---