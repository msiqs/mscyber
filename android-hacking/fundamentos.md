Mobile hacking vem crescendo cada vez mais no que concerne estudos especializados, demanda e interesse por parte dos profissionais e entusiastas de cyber. Pela abordagem inicial ser diferente de um web ou infra, algumas pessoas constroem sob si mesmas barreiras que às impedem de se aprofundar nessa "nova tendência". Frases como "se é louco, mobile não é pra mim não..." são comunmente ditas por integrantes da comunidade. Dessa forma, esse artigo tem como objetivo desmistificar o mobile hacking.

# O Android.

Para atacar o Android, você precisa entender que ele é, na essência, um Linux paranoico... 

## Arquitetura e Segurança do Android
Antes de atacar um reino, precisamos saber como funcionam as coisas do lado de dentro das muralhas... O android possui possui componentes que são divididos em seis camadas, como ilustrado abaixo.

![Figura 1 - Ilustração da arquitetura](image.png)

### System Apps

A camada de System Apps representa a interface direta entre o usuário e o dispositivo, mas para um atacante, ela é a porta de entrada privilegiada. Enquanto a exploração remota de Kernel e HAL exige exploits complexos e específicos de hardware, os System Apps são softwares massivos, propensos a erros de lógica e falhas humanas.

Estatisticamente, é nesta camada que encontramos as vulnerabilidades mais acessíveis. A distinção crucial aqui não é apenas funcional, mas estrutural e de privilégios.

**System Apps x User Apps:**

Os **System Apps** representam a camada privilegiada do ecossistema, residindo em diretórios protegidos como /system/app ou /system/priv-app. Diferente dos softwares comuns, eles são instalados estaticamente junto com a imagem do sistema operacional e habitam uma partição Read-Only, o que significa que não podem ser removidos ou modificados sem acesso Root, oferecendo um vetor excelente para persistência de malware. O valor crítico para a segurança ofensiva, no entanto, reside nos seus privilégios: estes aplicativos frequentemente operam com permissões de nível signature ou system, o que lhes confere é, na maioria dos casos, sinônimo de uma escalação de privilégio bem-sucedida. 

Em contraste, os **User Apps** são os aplicativos instalados dinamicamente pelo usuário e alocados no diretório /data/app. Sob a ótica da arquitetura de segurança, eles operam sob restrições severas: rodam confinados na sandbox padrão do Android, cada um com seu próprio User ID isolado. Isso cria um mecanismo de contenção de danos; se um aplicativo de usuário for explorado, o impacto **geralmente** se restringe ao vazamento de dados daquele contexto específico, sem comprometer a integridade do Kernel ou do sistema operacional. Além disso, não possuem a característica de imutabilidade, podendo ser desinstalados completamente a qualquer momento, o que torna a manutenção de acesso (persistência) muito mais desafiadora para um atacante.

### Java API Framework

O Java API Framework é a interface exposta aos desenvolvedores. Para Offensive Security, é o campo de batalha da engenharia reversa dinâmica. É aqui que manipulamos a lógica de funcionamento dos aplicativos através de instrumentação e exploramos falhas lógicas na gestão de permissões e intents.

### Native C/C++ Libraries

Abaixo da camada confortável do Java Framework, reside um conjunto robusto de bibliotecas escritas em C e C++. A existência dessa camada é justificada por uma única palavra: **performance**. O Android delega tarefas pesadas, como renderização gráfica (OpenGL/Vulkan), processamento de áudio, criptografia (SSL) e renderização de páginas web (WebKit), para o código nativo, pois o Java seria lento demais para executá-las em tempo real.

Em segurança ofensiva, esta camada representa o "Velho Oeste". Diferente do Java, que possui um Garbage Collector para gerenciar a memória automaticamente e prevenir erros grosseiros, o C/C++ coloca a responsabilidade do gerenciamento de memória inteiramente nas mãos do desenvolvedor. E, como a história nos ensina, desenvolvedores cometem erros. Um erro aqui não apenas trava o aplicativo; ele abre brechas para sobrescrever a memória do processo, sequestrar o fluxo de execução e obter acesso direto ao sistema, muitas vezes ignorando completamente as proteções da camada superior.

ã õ à À





