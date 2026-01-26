# As Regras do Jogo: O Modelo de Segurança

Agora que conhecemos a arquitetura, precisamos entender as defesas. O Android não foi desenhado para ser amigável com quem quer fuçar suas entranhas. Ele foi desenhado para manter cada aplicativo em uma cela solitária.

Para um atacante, entender essas regras é vital, pois a maioria dos "hacks" em mobile não envolve apenas explorar um bug, mas sim encadear falhas para escapar dessas restrições.

## 1. Application Sandbox: A Solitária

Lembra que chamamos o Android de "Linux Paranoico"? A Sandbox é a manifestação física dessa paranoia.

No Linux tradicional, se eu rodo dois programas como o usuário `matheus`, ambos têm os mesmos privilégios e podem acessar os arquivos um do outro. No Android, isso não existe.
O sistema atribui um **UID (User ID) único** para cada aplicativo no momento da instalação.

* **A Barreira:** O App do Banco (UID 1001) não consegue ler os arquivos do App de Notas (UID 1002), porque, para o Kernel, eles são usuários completamente diferentes.
* **A Visão do Atacante:** Nossa meta inicial em qualquer pentest é, frequentemente, conseguir execução de código dentro do contexto (UID) do aplicativo alvo. Só estando "dentro" da Sandbox dele é que podemos roubar seus segredos (bancos de dados SQLite, SharedPreferences, Tokens). De fora, a Sandbox nos bloqueia (a menos que tenhamos Root).

## 2. O Modelo de Permissões: O Porteiro

Como os aplicativos estão isolados, eles não podem fazer nada interessante (usar a câmera, ler contatos, acessar a internet) sem pedir permissão explicita.

O arquivo `AndroidManifest.xml` é onde o aplicativo lista seus desejos.
* **Protection Levels:** O Android classifica essas permissões por risco. Permissões `normal` (como internet) são dadas automaticamente. Permissões `dangerous` (como GPS ou Contatos) exigem o consentimento do usuário.
* **O Vetor de Ataque:** Desenvolvedores preguiçosos ou frameworks mal configurados muitas vezes pedem permissões excessivas (*Over-privileged apps*). Se um app de lanterna pede acesso aos seus contatos, isso é uma superfície de ataque. Além disso, apps podem definir **permissões customizadas** para expor suas próprias funcionalidades. Se elas não forem configuradas corretamente (ex: `android:protectionLevel="signature"`), qualquer outro app malicioso instalado no celular pode sequestrar essas funções.

## 3. Code Signing: A Identidade

No Android, todo APK deve ser assinado digitalmente com um certificado do desenvolvedor antes de ser instalado. Isso garante a integridade e a autoria do código.

* **A Regra:** O Android não permite instalar uma atualização de um app se a assinatura digital não bater com a da versão já instalada.
* **O Obstáculo para o Hacker:** Quando fazemos engenharia reversa de um APK para injetar um malware ou modificar uma função (Tampering), nós quebramos a assinatura original. Para reinstalar o app modificado no dispositivo, somos obrigados a reassiná-lo com nossa própria chave. Isso cria um problema: o app modificado perde o acesso aos dados do app original (devido à mudança de assinatura) e pode ser bloqueado por mecanismos de proteção do Google Play Protect.