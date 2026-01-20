ã õ à À


Mobile hacking vem crescendo cada vez mais no que concerne estudos especializados, demanda e interesse por parte dos profissionais e entusiastas de cyber. Pela abordagem inicial ser diferente de um web ou infra, algumas pessoas constroem sob si mesmas barreiras que às impedem de se aprofundar nessa "nova tendência". Frases como "se é louco, mobile não é pra mim não..." são comunmente ditas por integrantes da comunidade. Dessa forma, esse artigo tem como objetivo desmistificar o mobile hacking.

# O Android.

Para atacar o Android, você precisa entender que ele é, na essência, um Linux paranoico... 

## Arquitetura e Segurança do Android
Antes de atacar um reino, precisamos saber como funcionam as coisas do lado de dentro das muralhas... O android possui possui componentes que são divididos em seis camadas, como ilustrado abaixo.

![Figura 1 - Ilustração da arquitetura](image.png)

### System Apps

A camada de System Apps é a ponte entre o usuário e o controle total. Enquanto o Kernel e o HAL são difíceis de explorar remotamente, os System Apps são softwares complexos, cheios de funcionalidades e, estatisticamente, cheios de erros humanos de programação. Eles são o alvo preferencial para ataques locais e exploração de falhas lógicas. E aqui, existe um parêntese a ser feito, system apps não são a mesma coisa que user apps, estes, ficam até em locais diferentes dentro do android, sendo syste
/app ou system/priv-app (system apps) e /data/app (user apps).

**System Apps x User Apps:**

Os user apps rodam com privilégios limitados. Se comprometidos, o dano geralmente se restringe aos dados do próprio usuário. Podem ser desinstalados facilmente. Já os system apps são instalados na partição read-only. Frequentemente possuem permissões de nível de sistema que um app comum jamais conseguiria. Não podem ser desinstalados sem acesso root.





