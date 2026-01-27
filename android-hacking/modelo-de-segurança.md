# As Regras do Jogo: O Modelo de Segurança

O Android não confia em ninguém. Entender as defesas nativas é vital, pois 90% do trabalho de um exploit moderno é fazer o bypass dessas mitigações.

## 1. Application Sandbox & UIDs

O Android isola apps usando UIDs do Linux. O App A (UID 10001) não pode ler arquivos do App B (UID 10002).
* **Ataque:** Nossa meta é conseguir RCE dentro do contexto do app alvo para herdar o UID dele e ler seus dados privados `/data/data/com.alvo/`.

## 2. Permissões e Scoped Storage

Antigamente, a permissão `READ_EXTERNAL_STORAGE` dava acesso total ao cartão SD.
A partir do **Android 10+**, o Google introduziu o **Scoped Storage**.

* **A Regra:** Mesmo com permissão de leitura, um app só vê seus próprios arquivos e arquivos de mídia públicos. Ele **não consegue** mais ler a pasta de downloads ou arquivos de outros apps soltos no armazenamento.
* **Impacto no Hacking:** Roubar arquivos via Path Traversal ou Directory Listing ficou muito mais difícil. Agora focamos em explorar FileProviders mal configurados para contornar isso.

## 3. Code Signing & Repackaging

Todo APK deve ser assinado. O Android usa esquemas de assinatura evolutivos:
* **v1 (JAR Signing):** Verifica arquivos individuais.
* **v2/v3/v4 (Full APK Signature):** Verifica o bit-a-bit do arquivo APK inteiro.

* **O Obstáculo:** Se tentarmos modificar um APK e recompilar, a assinatura v2/v3 quebra instantaneamente. O Android rejeita a instalação.
* **A Solução:** Precisamos remover a assinatura original `META-INF`, modificar o código, e **re-assinar** com nossa própria chave usando `apksigner`. Porém, ao mudar a assinatura, perdemos acesso aos dados antigos do app e quebramos integrações com Google Maps/Facebook Login que validam o hash do certificado.

## 4. SELinux: O Guarda-Costas

O **SELinux** (Security-Enhanced Linux) atua em modo **MAC** (Mandatory Access Control). Ele bloqueia ações baseadas em *políticas*, não apenas em quem você é.

* **Cenário Real:** Mesmo se você ganhar Root, o SELinux pode impedir que seu shell reverso acesse a câmera ou injete código em outro processo.
* **Bypass:** Em ambientes de teste, usamos `setenforce 0` para colocar o SELinux em modo Permissivo. Em exploits reais, precisamos encontrar falhas no kernel para desativar essa política em tempo de execução.

> **Hint: Flags de Depuração**
> Verifique sempre o `AndroidManifest.xml`. Se `android:debuggable="true"`, o jogo acabou. Você pode conectar o JDWP (Java Debug Wire Protocol) e ter uma shell dentro do app sem precisar de nenhum exploit.