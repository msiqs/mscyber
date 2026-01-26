# Crypto-Breaker

**Versão Atual:** v1.1 (Modular Edition)
**Linguagem:** Python 3.8+
**Repositório:** [github.com/msiqs/crypto-breaker](https://github.com/msiqs/crypto-breaker)

---

## O Conceito

O **Crypto-Breaker** é um framework de criptoanálise ofensiva desenvolvido para preencher uma lacuna deixada pelas ferramentas tradicionais como Hashcat e John The Ripper.

Enquanto as ferramentas padrão focam em **potência bruta** (GPU) para quebrar hashes conhecidos, o Crypto-Breaker foca em **ataques baseados em lógica** e cenários complexos de ofuscação frequentemente encontrados em Malwares e CTFs avançados.

A ferramenta foi desenhada para automatizar o processo de engenharia reversa de payloads que utilizam derivações de chave customizadas (KDFs) ou múltiplas camadas de encapsulamento.

## Principais Diferenciais

### 1. Análise Recursiva (Efeito Matryoshka)
A maior inovação da ferramenta é seu motor de detecção recursiva. Ao quebrar uma camada de criptografia, o Crypto-Breaker analisa o payload resultante.
* Se for um arquivo **GZIP/ZIP**, ele descomprime.
* Se for **Base64/Hex**, ele decodifica.
* Se for um **JSON**, ele extrai os campos.

O processo se repete em loop até que o texto plano final seja revelado, automatizando horas de trabalho manual de "descascar" payloads.

### 2. Arquitetura Modular (v1.1)
O projeto nasceu como um script monolítico, mas evoluiu para uma aplicação robusta seguindo princípios de **Software Engineering**.
* **Core Engine:** Separado da lógica de ataque, gerenciando apenas I/O e orquestração.
* **Worker Pools:** Utiliza `multiprocessing` real para contornar o GIL (Global Interpreter Lock) do Python, garantindo uso de 100% da CPU em todos os núcleos.
* **Plugin System:** Permite que o operador escreva pequenos scripts em Python para atacar lógicas proprietárias (ex: `MD5(Reverse(Password) + Salt)`) sem alterar o código fonte da ferramenta.

### 3. Foco em "Looting" (Batch Mode)
Desenvolvido pensando em cenários de pós-exploração, onde o atacante extrai um banco de dados contendo milhares de credenciais cifradas com vetores de inicialização (IVs) e Salts diferentes. O modo Batch processa arquivos JSON/CSV massivos de forma assíncrona.

## Stack Tecnológica

O projeto foi construído utilizando bibliotecas nativas e de baixo nível para garantir performance e controle granular sobre a memória.

* **Linguagem:** Python 3
* **Criptografia:** `pycryptodome` (AES, ChaCha20)
* **Concorrência:** `multiprocessing` (Process-based parallelism)
* **Design Patterns:** Strategy Pattern (para seleção de algoritmos) e Factory (para geração de workers).

## Exemplo de Uso

O Crypto-Breaker utiliza uma interface CLI moderna. Abaixo, um exemplo de ataque a um arquivo criptografado com AES-CBC utilizando uma wordlist:

```bash
python3 main.py aes -f secret_data.enc -w rockyou.txt -m CBC --iv "BASE64_IV"
```
```Plaintext
[+] Target Loaded: secret_data.enc
[+] Mode: AES-CBC | Workers: 12
[+] Cracking...
---------------------------------------------------
[SUCCESS] Password Found: "admin123"
[INFO] Payload detected as GZIP. Decompressing...
[INFO] Inner Payload is JSON. Saving to secret_data.json
---------------------------------------------------
```

## Roadmap 

[x] Refatoração Modular (v1.1)

[ ] Implementação de GPU Offloading (CUDA via Numba)

[ ] Módulo de análise estática de entropia