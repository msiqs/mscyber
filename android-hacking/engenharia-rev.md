# Engenharia Reversa: O Raio-X do APK

Após entender sobre o acesso ao dispositivo via ADB, precisamos entender o alvo. No mundo mobile, diferente da web onde o código fonte (HTML/JS) é entregue aberto ao navegador, o aplicativo vem empacotado e compilado.

A **Engenharia Reversa** é a arte de desmontar esse pacote para entender a lógica de negócio, encontrar segredos *hardcoded* e mapear a superfície de ataque sem precisar executar o aplicativo.

## 1. Anatomia de um APK

Antes de abrir ferramentas, você precisa entender o que está atacando. Um arquivo `.apk` (Android Package) nada mais é do que um arquivo **ZIP** renomeado. Se você mudar a extensão de `app.apk` para `app.zip`, você consegue abri-lo.

Dentro dele, a estrutura padrão que interessa a um hacker é:

* **`AndroidManifest.xml`:** O mapa do tesouro. (Importante: No APK, ele está em formato binário ilegível, precisaremos de ferramentas para decodificá-lo).
* **`classes.dex`:** O cérebro. Aqui está todo o código Java/Kotlin compilado no formato Dalvik Executable.
* **`lib/`:** O músculo. Bibliotecas nativas (.so) escritas em C/C++ para arquiteturas específicas (ARM, x86).
* **`res/` e `resources.arsc`:** A pele. Imagens, layouts e strings de texto.

## 2. A Ferramenta Rei: JADX

Para análise estática de código Java, a ferramenta padrão da indústria é o **JADX**. Ele atua como um descompilador, pegando o bytecode `classes.dex` e tentando reconstruí-lo para código Java legível.

### Por que usamos o JADX?
Ele nos dá uma interface gráfica (JADX-GUI) similar a uma IDE, como o VS Code ou IntelliJ, permitindo clicar em funções, seguir variáveis e buscar por strings em todo o projeto.

## 3. O Que Estamos Caçando? (Static Analysis Checklist)

Ao abrir um APK no JADX, não vamos ler 100.000 linhas de código linha por linha. Nós fazemos buscas por classes, arquivos, strings muito específicas que geralmente nos dão um norte ou entregam por si só uma vulnerabilidade logo de cara.

### A. O Manifesto (`AndroidManifest.xml`)
A primeira parada obrigatória. O JADX reconstrói o XML binário para texto.
* **Busca:** Procure por `exported="true"`.
* **Alvo:** Activities que podem ser abertas sem login, Broadcast Receivers que aceitam mensagens de qualquer um, ou Content Providers desprotegidos.

### B. Segredos Hardcoded
Desenvolvedores frequentemente cometem o erro de deixar chaves de API e credenciais no código para facilitar o desenvolvimento e esquecem de tirar.
* **Busca:** Strings como `api_key`, `Authorization`, `Bearrer`, `AWS`, `firebase`, `password`, `crypt`, `aes`.
* **Arquivo `res/values/strings.xml`:** Muitas vezes as chaves não estão no código Java, mas escondidas nesse arquivo de recursos.

### C. Lógica de Criptografia
Se o app usa criptografia, ele precisa da chave em algum lugar.
* **Busca:** Classes que importam `javax.crypto` ou `java.security`.
* **Alvo:** Procure por gerações de chaves simétricas (AES) onde a Key ou o vetor de inicialização (IV) são estáticos ou gerados de forma previsível.

### D. Endpoints de API
Para atacar o backend, precisamos saber quais URLs o app acessa.
* **Busca:** `https://`, `.com/api`, `/v1/`.
* **Objetivo:** Mapear rotas da API que não são visíveis durante a navegação normal do aplicativo, como funcionalidades de admin, rotas de teste, legados...

## 4. O Submundo: Java vs Smali

O JADX nos mostra **Java**, que é fácil de ler. Mas o Android não roda Java, ele roda Bytecode. Quando precisarmos *modificar* o aplicativo (para remover uma verificação de root ou injetar um malware), não podemos editar o Java do JADX, pois ele é apenas uma "tradução aproximada".

Nós precisaremos editar o **Smali**.
* **Smali** é a representação legível do bytecode DEX.
* Enquanto o Java é alto nível (`if (senha == "123")`), o Smali é baixo nível e trabalha com registradores (`if-eq v0, v1, :cond_0`).

> **Nota do Atacante:** Análise Estática é leitura. Para escrita e modificação, usaremos outra ferramenta chamada `Apktool` no futuro. Por enquanto, foque em entender o código que você vê.