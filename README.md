##Relatório para Geração de Dados de Ataques de DDoS em Ambiente Emulado

**DAVI BRITO, MICHELE NOGUEIRA**

### I. Introdução

Os ataques de negação de serviço distribuído (*Distributed Denial of Service* - DDoS) têm aparecido com mais recorrência e, além disso, com maior complexidade e em diversos ambientes que incluem, por exemplo, dispositivos da Internet das Coisas (*Internet of Things* - IoT).

A partir disso, a fim de evitar danos e perdas, torna-se imprescindível desenvolver técnicas cada vez mais sofisticadas para combater esse tipo de ataque. Contudo, para construir e treinar soluções contra ataques de DDoS, é necessário possuir dados condizentes com uma situação próxima da realidade. Uma característica desejada para esses dados seria, por exemplo, a existência de tráfego prévio ao ataque, preferencialmente abrangendo indicativos de ataque no futuro, visto que, no caso de um algoritmo de predição, seria algo benéfico. 

Ademais, é interessante que tráfegos de rede utilizados para treinamentos das soluções incluam dispositivos da IoT. Dispositivos como esses são cada vez mais comuns nas redes atuais e servem, inclusive, diversas vezes como vetor para ataques DDoS.

Com isso, criou-se a proposta de gerar um tráfego de ataques de DDoS próprio, que fosse fácil de reproduzir e que buscasse ser o mais próximo possível de um cenário real, utilizando, também, dispositivos da IoT. Para atingir esses objetivo, foi utilizado um software de emulação de redes, em conjunto com outras ferramentas voltadas à geração e captura do tráfego.

A metodologia utilizada para geração do conjunto de dados é melhor descrita nas seções seguintes, estando dividida da seguinte forma: a Seção II apresentará como instalar e configurar o ambiente de emulação. A Seção III tratará da geração de tráfego de dispositivos da IoT simulados. A Seção IV apresentará a geração de tráfego simulado de dispositivos comuns e dispositivos infectados. A Seção V abordará a condução do experimento completo, assim como a captura dos dados. Por fim, a Seção VI concluirá este relatório.


### II. Ambiente de Emulação

O ambiente de emulação é o espaço onde todo o experimento será orquestrado, conduzido e capturado, por fim. Por meio dele é possível implementar a topologia pensada para o experimento, obtendo, assim, uma rede emulada, customizável e escalável. O ambiente de emulação utilizado nesta metodologia é o [Common Open Research Emulator](https://coreemu.github.io/core/ "Common Open Research Emulator").

O CORE é um emulador de redes capaz de criar representações de dispositivos reais utilizando o *kernel* Linux da máquina real. Na prática, o CORE se baseia em cima do conceito de *Container-Based Emulation* (CBE), ou seja, cada nó criado dentro da rede consiste em um contêiner leve que está inserido dentro da máquina principal. A ferramenta possui uma interface gráfica simples, o que auxilia na definição de topologias e configurações. Essa característica foi crucial para sua escolha como ambiente do experimento.

Um aspecto importante de se destacar é que foi utilizada uma [máquina virtual](https://encurtador.com.br/jrvFQ "máquina virtual") (senha: core) com uma versão mais antiga do CORE para realizar a criação do ambiente emulado. Essa escolha foi feita devido a problemas relacionados à nova versão da ferramenta e sua instalação local, assim como uma maior familiaridade com a versão legado. Possíveis testes com versões mais novas podem ser feitos para adequação da metodologia aqui utilizada, mas isso não será abordado neste relatório.

Ao inicializar o CORE, é possível visualizar na sua interface gráfica as opções relacionadas à definição dos dispositivos, edições meramente visuais e início da emulação (Figura 1). O CORE possui alguns tipos de dispositivos pré-configurados, como computadores, roteadores, *switches*, *hubs* e outras interfaces. Mudanças nos nomes dos dispositivos e seus endereços IP podem ser realizadas. A partir da disposição dos nós, é possível interligá-los rapidamente com a opção "*link tool*" da ferramenta. Ainda é permitido alterar configurações dos enlaces específicos utilizados, mudando aspectos como largura de banda, latência, taxa de perda, entre outros.

[![Figura 1. Interface do CORE](https://github.com/mentoredproject/WP2-metodologia-cria-o-dados/blob/main/imgs/core.png "Figura 1. Interface do CORE")](https://github.com/mentoredproject/WP2-metodologia-cria-o-dados/blob/main/imgs/core.png "Figura 1. Interface do CORE")

Por fim, definida a topologia e os enlaces, é possível iniciar a emulação a partir da opção "\textit{start the session}". Com isso, é possível interagir com cada dispositivo da rede, abrindo um terminal para cada um deles a partir de um clique duplo. Além disso, o CORE permite visualizar informações rápidas de cada nó e executar um comando específico em todos eles.


### III. Tráfego IoT

A obtenção de tráfego de dispositivos da IoT foi feita por meio de softwares de simulação. Utilizar dispositivos reais pode ser mais benéfico em relação à fidelidade do tráfego, contudo, utilizar simulação se torna menos custoso e de maior facilidade de implantação, captura e customização.

Dentre as ferramentas de geração de tráfego IoT simulado pesquisadas, foi selecionada a ferramenta [IoT-Flock](https://github.com/ThingzDefense/IoT-Flock "IoT-Flock"). A escolha se deu por ela permitir criar casos de uso e por conter perfis pré-definidos para alguns dispositivos IoT. O software também permite que os dispositivos sejam configurados como entidades de ataque (TCP *flood*, UDP *flood* etc). Para funcionar, os dispositivos IoT têm que ser configurados para realizar a conexão com um *broker* ou servidor. O IoT-Flock suporta os protocolos CoAP e MQTT para trafegar os dados.

Um ponto importante a ser mencionado é que houveram diversas dificuldades para instalação local da ferramenta, visto que ela não possui atualizações recentes. Esse problema só foi solucionado com a instalação de uma máquina virtual disponibilizada pelos [próprios desenvolvedores](https://encurtador.com.br/pBINP "próprios desenvolvedores") (senha: iot123)}. A partir da utilização dela, cria-se um problema: como integrar o tráfego IoT gerado com o restante da emulação (que acontece em outra máquina virtual)? A solução pensada envolve a captura do tráfego do IoT-Flock e sua posterior replicação dentro do CORE com o uso da ferramenta Tcpreplay. O Tcpreplay é capaz de replicar todo o tráfego contido em um arquivo PCAP em tempo real, viabilizando essa solução.

Após iniciada a máquina virtual, é necessário acessar a pasta que contém o programa. Em seguida, a interface gráfica deve ser aberta para configuração dos casos de uso e dispositivos. Os comandos abaixo explicitam como fazer isso.

`cd ~/Desktop/IoTFlock`

`sudo ./IoTFlock-GUI`

Em seguida, clique na opção "*Add New UseCase*" e digite o nome desejado. Agora clique em "*Edit*" e "*Add New Device*" para criar e configurar os dispositivos IoT. É possível selecionar um dos *templates* pré-definidos de comportamento do dispositivo (sensores de temperatura, movimento, fumaça etc.). É importante que o endereço IP escolhido esteja na faixa da rede local da máquina (192.168.0.0/24). Ademais, selecione o protocolo MQTT e vá para suas configurações.

Neste momento, é preciso definir o endereço IP do *broker* MQTT para 127.0.0.1 e desativar a opção "*Broker Authentication*". Por fim, é necessário configurar o tipo de dado a ser gerado e sua frequência. Os campos "*Topic*", "*Data Profile*" e "*Time Profile*" possuem essas configurações, sendo que a frequência de geração pode ser periódica ou aleatória. Por fim, clique em "*Add*" para adicionar o comportamento do dispositivo. A Figura 2 demonstra um exemplo de configuração correta. Além disso, a Figura 3 traz um panorama geral da configuração da ferramenta.

![Figura 2. Configuração de dispositivo IoT](https://github.com/mentoredproject/WP2-metodologia-cria-o-dados/blob/main/imgs/iotconfig.png "Figura 2. Configuração de dispositivo IoT")

![Panorama de configuração do IoT-Flock](https://github.com/mentoredproject/WP2-metodologia-cria-o-dados/blob/main/imgs/iotflockProcess.png "Panorama de configuração do IoT-Flock")

Uma vez criados todos os dispositivos, é necessário salvar o arquivo XML de configuração por meio da opção "*Save XML*", na aba de casos de uso. Esse arquivo é responsável por guardar as informações que o *console* do IoT-Flock precisa se conectar e gerar o tráfego. Para fazer isso, você deve utilizar o executável "*IoTFlock-Console*" e passar o arquivo XML como parâmetro. O comando exato está descrito abaixo.


`cd ~/Desktop/IoTFlock`

`sudo ./IoTFlock-Console <arquivo>`


A partir disso, o software se conectará ao *broker* MQTT e começará a gerar tráfego. Desse modo, agora é necessário capturá-lo, uma vez que este deverá ser replicado durante a execução da emulação com CORE. É possível fazer essa captura via Wireshark, Tcpdump ou outras ferramentas relacionadas. O tráfego pode ser visualizado na interface de *loopback* (lo) e filtrado pelo protocolo TCP ou pelo endereço IP do dispositivo IoT. Assim, restará somente a comunicação do IoT-Flock.

Por fim, ao capturar todo o tráfego IoT desejado, é preciso exportar o arquivo PCAP para dentro da máquina virtual da ferramenta CORE. Esse arquivo será utilizado posteriormente para replicação de tráfego em tempo real com o uso da ferramenta Tcpreplay. Para o caso em que se deseje alterar os endereços IP presentes na captura, foi desenvolvido um código simples para fazer a [modificação do PCAP](https://encurtador.com.br/nBGLS "modificação do PCAP").

### IV. Tráfego Comum e de Ataque

Diferentemente da estratégia utilizada para gerar o tráfego de dispositivos da IoT, o tráfego de dispositivos comuns e dispositivos infectados será gerado dentro do ambiente de emulação. Inicialmente, foi escolhido o ataque DDoS do tipo SYN *flood*, devido à sua simplicidade de implementação. A ferramenta escolhida para essa tarefa é o [hping3](https://www.kali.org/tools/hping3 "hping3"). A ferramenta consiste em um gerador de tráfego de propósito geral, possuindo como uma de suas funcionalidades o envio acelerado de pacotes. Após instalar a ferramenta, pode-se abrir o terminal de um dos nós emulados e executar o  comando abaixo para realizar um ataque do tipo SYN *flood*.

`hping3 --flood -S -q <ip> &`


Para a geração de tráfego comum, foram pesquisadas ferramentas de propósito geral, uma vez que não foram encontradas soluções com esse propósito específico. O software [Ostinato](https://ostinato.org/ "Ostinato") foi testado, contudo, por se tratar de uma ferramenta paga, seu uso foi descartado frente a outras alternativas. Portanto, escolheu-se utilizar novamente o hping3 para geração desse tipo de tráfego, justamente por ter propósito geral. Abordagens mais sofisticadas que envolvam, por exemplo, o uso de serviços com protocolos como HTTP, FTP e SMTP podem ser desenvolvidas para tornar o tráfego mais próximo da realidade. O comando descrito abaixo gera um tráfego TCP com a flag SYN e intervalo de envio de um segundo.


`hping3 -S -q <ip> &`


### V. Condução e Captura

A partir da definição do ambiente de emulação, topologia da rede, tipos de comunicações e do planejamento geral do conjunto de dados, é necessário, agora, conduzir todo o experimento em tempo real. Essa etapa dependerá muito de como foi projetado o experimento como um todo, com fatores como o tempo de duração da captura, quantidade de dispositivos, quantidade de alvos do ataque e duração do ataque variando em cada contexto. Portanto, é imprescindível realizar as adaptações necessárias nas abordagens e códigos aqui disponibilizados.

Com a rede emulada em execução, a captura de todo o tráfego deve ser iniciada antes de qualquer tipo de tráfego ser gerado. Para realizar essa captura, utilizou-se o software Tcpdump, visto que sua exigência de recursos computacionais é menor em relação ao Wireshark, por exemplo. Para a interface de captura, utilizou-se uma com nome "b.2.46", presente no teste da Figura 1, por ser possível visualizar todo o tráfego da rede. Para capturar o tráfego durante dez minutos, gerando somente um arquivo de nome "captura", utilizou-se o comando abaixo.


`sudo tcpdump -i b.2.46 -G 600 -W 1 -w captura.pcap `


Após iniciar a captura, foram utilizados [scripts](https://encurtador.com.br/cmzNY "scripts") para execução automática da geração de tráfego. Por meio deles, o tráfego IoT passava a ser replicado com tcpreplay, além dos comandos com hping3 serem executados. Após um certo tempo pré-definido, os processos que envolviam a geração de tráfego eram encerrados e a captura terminava. É necessário alterar o script de acordo com a especificação desejada para o experimento.Por fim, o dataset experimental criado utilizando as ferramentas selecionadas foi disponibilizado no endereço: [https://shre.ink/rfwg](https://shre.ink/rfwg "https://shre.ink/rfwg"). 

### VI. Conclusão


Este relatório busca facilitar o processo de geração de dados em contextos de ataques de DDoS, com o intuito de utilizá-los para treinamento e teste de soluções envolvendo esse tipo de ataque. O processo aqui apresentado é completamente expansível e modificável, no caso do surgimento de soluções mais eficientes, práticas e próximas da realidade. Novas ferramentas para geração de cada tipo de tráfego e, também, para criação de um ambiente emulado, por exemplo, podem ser testadas e utilizadas para o propósito aqui pretendido.
