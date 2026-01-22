ã õ à À


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

Os user apps rodam com privilégios limitados. Se comprometidos, o dano geralmente se restringe aos dados do próprio usuário. Podem ser desinstalados facilmente. Já os system apps são instalados na partição read-only. Frequentemente possuem permissões de nível de sistema que um app comum jamais conseguiria. Não podem ser desinstalados sem acesso root. Já os system apps geralmente possuem alto nível de permissão de sistema, coisa que um app comum jamais teria. São exemplo de system app: System UI, Settings, etc... 

### Java API Framework

O Java API Framework é a interface exposta aos desenvolvedores. Para Offensive Security, é o campo de batalha da engenharia reversa dinâmica. É aqui que manipulamos a lógica de funcionamento dos aplicativos através de instrumentação e exploramos falhas lógicas na gestão de permissões e intents.



