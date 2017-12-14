#DOCKER (Container)#

##Benefícios##
* mais leve do que virtualização de máquinas
* velocidade deploy
* custo reduzido
* melhor controle sobre uso d cada recurso (CPU, Disco, Rede, ...)
* agilidade na hora de criar ou remover um container
* maior facilidade de trabalhar com diferentes versões de bibliotecas

##Conteúdo##
* Ciclo de vida Containers
* Comandos
* Baixar imagens (DockerHub e DockerStore)
* Volume
* Comunicação Containers
* Docker Compose (Orquestração)
* Micro-serviços
* Cloud

##O que é o Docker?##
O **Docker** nada mais é do que uma coleção de tecnologias para facilitar o deploy e a execução das nossas aplicações. A sua principal tecnologia é a **Docker Engine**, a plataforma que segura os containers, fazendo o intermédio entre com o sistema operacional.

Outras tecnologias do Docker que facilitam a nossa vida e que veremos neste curso são o **Docker Compose**, um jeito fácil de definir e orquestrar múltiplos containers; o **Docker Swarm**, uma ferramenta para colocar múltiplos docker engines para trabalharem juntos em um cluster; o **Docker Hub**, um repositório com mais de 250 mil imagens diferentes para os nossos containers; e a **Docker Machine**, uma ferramenta que nos permite gerenciar o Docker em um host virtual.

##Instalação Docker##
**Windows**: [Download](https://www.docker.com/docker-windows)
>* Pré-Requisitos: Win 10 Pro, 64bits e Virtualização Habilitada
>* *O que rola por baixo panos: Instala o Docker Engine dentro de uma micro VM (Alpine Linux) e exige que SO tenha HyperVisor (Virtualização)*

**macOS**: [Download](https://www.docker.com/docker-mac)
>* Pré-Requisitos: Modelo 2010+, OS X 10.11+, 4GB RAM, VirtualBox superior à 4.3.30
>* *O que rola por baixo panos: Instala o Docker Engine dentro de uma micro VM (Alpine Linux) e exige que SO tenha HyperKit (Virtualização)*

##Execução##
Abrir *terminal* e executar **docker version**

Para criar hello-world: **docker run <NOME_DA_IMAGEM>**

```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
ca4f61b1923c: Pull complete
Digest: sha256:be0cd392e45be79ffeffa6b05338b98ebb16c87b255f48e297ec7f98e123905c
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://cloud.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/engine/userguide/
```

##Comandos Básicos##
```
#listar containers ativos
docker ps

#listar id dos containers ativos
docker ps -q

#listar containers parados
docker ps -a

#baixando e executando imagem
docker run <IMAGE_NAME>

#baixando e executando imagem de um usuário (docker run dockersamples/static-site)
docker run <USER_NAME>/<IMAGE_NAME>

#parametros: -d nao trava seu terminal, -P container liberar porta maquina para acessar browser, --name deixa nomear a sua imagem
#ex 01: docker run -d -P --name meu-site dockersamples/static-site
#ex 02: docker run -d -p 12345:80 dockersamples/static-site (-p permite vc selecionar qual porta sua maquina vai atender na porta 80 do container)
#ex 03: docker run -d -P -e AUTHOR="Luis Garcia" dockersamples/static-site (-e cria variável ambiente dentro do container)

#trabalhar dentro container
docker run -it <CONTAINER_NAME>

#verificar portas em uso pelo container
docker port <CONTAINER_ID>

#start/stop container (-t, parametro de tempo espera execução)
docker start <CONTAINER_ID>
docker stop -t 0 <CONTAINER_ID>
docker stop -t 0 $(docker ps -q)

#remover container inativo
docker rm <CONTAINER_ID>

#remover tds containers inativos (stopped)
docker container prune

#remover imagens
docker rmi <IMAGE_NAME>
```

##Layered Filesystem##
As camadas das imagens baixadas (READ-ONLY) são compartilhadas, caso precise baixar mais de uma imagem que utilize uma camada que já tenha baixada localmente, ela será compartilhada. Porém é possível alterar a camada base do container criado (READ-WRITE).

##Volumes##
Containers são voláteis, logo pode adicionar e remover o container com certa frequência. Desta forma tudo o que for escrito no container será perdido. Sendo assim o uso de Volumes se faz necessário, pois voce cria ponteiro no seu container para um volume criado dentro do seu Docker Host.
```
#utilizar parametro -v para apontar a pasta do seu volume, sendo <PASTA_DOCKER_HOST>:<NOME_VOLUME>
docker run -it -v "C:\Users\lgarcia\Desktop:/var/www" ubuntu
```

É possível verificar como volume foi montado executando:
```
docker inspect <ID_CONTAINER>
```

Isso possibilita vc usar containers especificos para determinadas linguagens, por exemplo:
* Container Node: pode ter neste container node/npm instalado e criar um volume de dados com codigos node onde poderá ser executado somente neste container
* Container Java: pode ter neste container configurado jdk/jvm e executar códigos java dentro do seu volume, sem a necessidade de cada desenvolvedor replicar ambiente.

```

cd Desktop/volume-exemplo/

pwd

#parametros: -d (detached), -p (portas), -v (volume), -w (work-directory)
docker run -d -p 8080:3000 -v "$(pwd):/var/www" -w "/var/www" node npm start
```

##Dockerfile##
Criar o arquivo Dockerfile (default) para criar imagem:
```
docker run -p 8080:3000 -v "$(pwd):/var/www" -w "/var/www" node npm start
```

Dockerfile baseado no comando acima:
```
FROM node:latest
MAINTAINER Luis Garcia
ENV NODE_ENV=production
ENV PORT=3000
COPY . /var/www
WORKDIR /var/www
RUN npm install
ENTRYPOINT ["npm", "start"]
EXPOSE $PORT
```

[Buildando a imagem] Como o nome do nosso Dockerfile é o padrão, poderíamos omitir esse parâmetro, mas se o nome for diferente, por exemplo node.dockerfile, é preciso especificar, mas vamos deixar especificado para detalharmos melhor o comando.
```
docker build -f Dockerfile -t lgarcia/node
```

Agora que já temos a imagem criada, podemos criar um container a partir dela:
```
docker run -d -p 8080:3000 douglasq/node
```

Mandar imagem para o DockerHub (Compartilhar):
* [DockerHub](https://hub.docker.com/)
* Login no DockerHub

Via linha de comando, faz o publish da imagem:
```
docker login

docker push <NOME_DA_IMAGEM>
```

Para baixar imagem:
```
docker pull <NOME_DA_IMAGEM>
```

##Networking no Docker##
Normalmente uma aplicação é composta por diversas partes, sejam elas o load balancer/proxy, a aplicação em si, um banco de dados, etc. Quando estamos trabalhando com containers, é bem comum separarmos cada uma dessas partes em um container específico, para cada container ficar com somente uma única responsabilidade.

Mas se temos uma parte da nossa aplicação em cada container, como podemos fazer para essas partes falarem entre elas? Pois para a nossa aplicação funcionar como um todo, os containers precisam trocar dados entre eles.

![Múltiplos Containers](/img/multiplos_containers.png)

A boa notícia é que no Docker, por padrão, já existe uma *default network*. Isso significa que, quando criamos os nossos containers, por padrão eles funcionam na mesma rede:

![Default Network](/img/default_network.png)

Agora, no primeiro container, vamos instalar o pacote iputils-ping para podermos executar o comando ping para verificar a comunicação entre os containers:
```
#rodar 2 containers de ubuntu
docker run -it ubuntu

#ja no terminal ubuntu
root@973feeeeb1df:/# hostname -i

#instala modulo de ping
apt-get update && apt-get install iputils-ping

#verifica ping do outro container
ping <IP_OUTRO_CONTAINER>
```

Na rede padrão do Docker, só podemos realizar a comunicação utilizando IPs, mas se criarmos a nossa própria rede, podemos "batizar" os nossos containers, e realizar a comunicação entre eles utilizando os seus nomes:

![Container Name](/img/container_name.png)

Isso não pode ser feito na rede padrão do Docker, somente quando criamos a nossa própria rede.

##Criando a nossa própria rede do Docker##
Então, vamos criar a nossa própria rede, através do comando docker network create, mas não é só isso, para esse comando também precisamos dizer qual driver vamos utilizar. Para o padrão que vimos, de ter uma nuvem e os containers compartilhando a rede, devemos utilizar o driver de bridge.
```
docker network create --driver bridge <NOME_REDE>

docker network ls

docker run -it --name meu-container-de-ubuntu --network minha-rede ubuntu
docker run -it --name segundo-ubuntu --network minha-rede ubuntu

#dentro do segundo ubuntu, ja eh possivel pingar pelo nome do container
root@00f93075d079:/# ping meu-container-de-ubuntu
```

##Testando imagem com cenario real##
```
docker pull douglasq/alura-books:cap05
docker pull mongo

docker run -d --name meu-mongo --network minha-rede mongo

docker run --network minha-rede --name minha-app -d -p 8080:3000 douglasq/alura-books:cap05
```

##Docker Compose##
Ao invés de subir todos esses containers na mão, o que vamos fazer é utilizar uma tecnologia aliada do Docker, chamada Docker Compose, feito para nos auxiliar a orquestrar melhor múltiplos containers. Ele funciona seguindo um arquivo de texto *YAML (extensão .yml)*, e nele nós descrevemos tudo o que queremos que aconteça para subir a nossa aplicação, todo o nosso processo de build, isto é, subir o banco, os containers das aplicações, etc.

![Container Name](/img/load_balancer.png)

Passos:
* Configurar NGINX (load balancer e arquivos estaticos)
* Configurar Aplicação
* Configurar Banco

Em todo arquivo de Docker Compose, que é uma espécie de receita de bolo para construirmos as diferentes partes da nossa aplicação, a primeira coisa que colocamos nele é a versão do Docker Compose que estamos utilizando:
```
version: '3'
```

Agora, começamos a descrever os nossos serviços. Um serviço é uma parte da nossa aplicação, lembrando do nosso diagrama temos NGINX, três Node, e o MongoDB como serviços. Logo, se queremos construir cinco containers, vamos construir cinco serviços, cada um deles com um nome específico (*docker-compose.yml*). 
```
version: '3'
services:
    nginx:
        build:
            dockerfile: ./docker/nginx.dockerfile
            context: .
        image: douglasq/nginx
        container_name: nginx
        ports:
            - "80:80"
        networks: 
            - production-network
        depends_on: 
            - "node1"
            - "node2"
            - "node3"

    mongodb:
        image: mongo
        networks: 
            - production-network

    node1:
        build:
            dockerfile: ./docker/alura-books.dockerfile
            context: .
        image: douglasq/alura-books
        container_name: alura-books-1
        ports:
            - "3000"
        networks: 
            - production-network
        depends_on:
            - "mongodb"

    node2:
        build:
            dockerfile: ./docker/alura-books.dockerfile
            context: .
        image: douglasq/alura-books
        container_name: alura-books-2
        ports:
            - "3000"
        networks: 
            - production-network
        depends_on:
            - "mongodb"

    node3:
        build:
            dockerfile: ./docker/alura-books.dockerfile
            context: .
        image: douglasq/alura-books
        container_name: alura-books-3
        ports:
            - "3000"
        networks: 
            - production-network
        depends_on:
            - "mongodb"

networks: 
    production-network:
        driver: bridge
```

Com o docker-compose.yml pronto, podemos subir os serviços, mas antes devemos garantir que temos todas as imagens envolvidas neste arquivo na nossa máquina. Para isso, dentro da pasta do nosso projeto, executamos o seguinte comando:
```
sudo docker-compose build
```

Com os serviços criados, podemos subi-los através do comando:
```
#subir serviços
docker-compose up

#caso queira rodar sem travar terminal com os logs
docker-compose up -d

#listar os serviços rodando
docker-compose ps

#parar serviços e remove
docker-compose down

#reiniciar os serviços
docker-compose restart

#help comandos
docker-compose --help
```

Esse comando irá seguir o que escrevemos no docker-compose.yml, ou seja, cria a rede, o container do MongoDB, os três containers da aplicação e o container do NGINX. Depois, são exibidos alguns logs, sendo que cada um dos containers fica com uma cor diferente, para podermos distinguir melhor.

E não é por que eles são serviços, que eles não tem um container por debaixo dos panos, então nós conseguimos interagir com os containers utilizando todos os comandos que já vimos no treinamento, por exemplo para testar a comunicação entre eles:
```
docker exec -it alura-books-1 ping alura-books-2
```

###Instalando o Docker Compose no Linux###
O Docker Compose não é instalado por padrão no Linux, então você instalá-lo por fora. Para tal, baixe-o na sua versão mais atual, que pode ser visualizada no seu GitHub, executando o comando abaixo:
```
sudo curl -L https://github.com/docker/compose/releases/download/1.15.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
```

Após isso, dê permissão de execução para o docker-compose:
```
sudo chmod +x /usr/local/bin/docker-compose
```

##Sobre Microserviços##

Já ouviu falar de Microserviços? Se já ouviu, pode pular a introdução abaixo e ir diretamente para a parte "Docker e Microserviços", senão continue comigo.

Uma forma de desenvolver uma aplicação é colocar todas as funcionalidades em um único "lugar". Ou seja, a aplicação roda em uma única instância (ou servidor) que possui todas as funcionalidades. Isso talvez é a forma mais simples de criar uma aplicação (também a mais natural) mas quando a base de código cresce alguns problemas podem aparecer.

Por exemplo, qualquer atualização ou bug fix necessita parar todo o sistema, buildar o sistema todo e subir novamente. Isso pode ficar demorado e lento. Em geral, quanto maior a base de código mais difícil será manter ela mesmo com uma boa cobertura de testes e as desvantagens não param por ai. Outro problema é se alguma funcionalidade possuir um gargalo no desempenho o sistema todo será afetado. Não é raro de ver sistemas onde relatórios só devem ser gerados à noite para não afetar o desempenho de outras funcionalidades. Outro problema comum é como ciclos de testes e build demorados (falta de agilidade no desenvolvimento), problemas no monitoramento da aplicação ou falta de escalabilidade. Enfim, o sistema se torna um legado pesado onde nenhum desenvolvedor gostaria de colocar a mão no fogo.

##Arquitetura de Microservicos##
Então a idéia é fugir desse tipo de arquitetura monolítica monstruosa e dividir ela em pequenos pedaços. Cada pedaço possui uma funcionalidade bem definida e roda como se fosse um "mini sistema" isolado. Ou seja, em vez de termos uma única aplicação enorme teremos várias instâncias menores que dividem e coordenam o trabalho. Essas instâncias são chamadas de Microserviços.

Agora fica mais fácil monitorar cada serviço específico, atualizá-lo ou escalá-lo pois a base de código é muito menor, e assim o deploy e o teste serão mais rápido. Podemos agora achar soluções específicas para esse serviço sem precisar alterar os demais. Outra vantagem é que um desenvolvedor novo não precisa conhecer o sistema todo para alterar uma funcionalidade, basta ele focar na funcionalidade desse microservico.

Importante também é que um microserviço seja acessível remotamente, normalmente usando o protocolo HTTP trocando mensagens JSON ou XML, mas nada impede que outro protocolo seja usado. Um microserviço pode usar outros serviços para coordenar o trabalho.

Repare que isso é uma outra abordagem arquitetural bem diferente do monolítico e por isso também é chamado de arquitetura de microserviços.

Por fim, uma arquitetura de Microserviços tem um grau de complexidade muito alta se comparada com uma arquitetura monolítica. Aliás, há aqueles profissionais que indicam partir para [uma arquitetura monolítica primeiro e mudar para uma baseada em microserviços depois](https://sdtimes.com/martin-fowler-monolithic-apps-first-microservices-later/), quando estritamente necessário.

##Docker e Microserviços##
Trabalhar com uma arquitetura de microserviços gera a necessidade de publicar o serviço de maneira rápida, leve, isolada e vimos que o Docker possui exatamente essas características! Com Docker e Docker Compose podemos criar um ambiente ideal para a publicação destes serviços.

O Docker é uma ótima opção para rodar os microserviços pelo fato de isolar os containers. Essa utilização de containers para serviços individuais faz com que seja muito simples gerenciar e atualizar esses serviços, de maneira automatizada e rápida.

##Docker Swarm##
Ok, tudo bem até aqui. Agora vou ter vários serviços rodando usando o Docker. E para facilitar a criação desses containers já aprendemos usar o Docker Compose que sabe subir e orquestrar vários containers. O Docker Compose é a ferramenta ideal para coordenar a criação dos containers, no entanto para melhorar a escalabilidade e desempenho pode ser necessário criar muito mais containers para um serviço específico. Em outras palavras, agora gostaríamos de criar muitos containers aproveitando várias máquinas (virtuais ou físicas)! Ou seja, pode ser que um microserviços fique rodando em 20 containers usando três máquinas físicas diferentes. Como podemos facilmente subir e parar esses containers? Repare que o Docker Compose não é para isso e por isso existe uma outra ferramenta que se chama o Docker Swarm (que não faz parte do escopo desse curso).

Docker Swarm facilita a criação e administração de um cluster de containers.

##Docker Cloud##
Entre os serviços na nuvem, podemos citar o Amazon Web Services, DigitalOcean e o Microsoft Azure, em que alugamos um servidor e hospedamos a nossa aplicação. Então, se alugarmos um servidor de qualquer uma dessas ou outras empresas, podemos pedir para subir uma máquina virtual, que provavelmente rodará um Linux, em que podemos instalar o Docker e fazer a nossa aplicação funcionar.

Mas isso requer um trabalho manual, pois devemos ir até o AWS, alugar o servidor, criar a máquina, abrir as suas portas, instalar um sistema operacional, todo um custo para subirmos as partes inferiores do nosso sistema.

Por isso a Docker, Inc. criou uma solução para nos poupar desse trabalho, que é o Docker Cloud, uma das tecnologias da empresa, que se conecta a algum dos serviços citados, e fica responsável de gerenciar essa parte de instalar a máquina virtual, subir um Linux e instalar o Docker, além de subir os nossos containers para nós:
![Docker Cloud](/img/docker_cloud.png)

###Pré-requisitos###
> * Conta criada no DockerCloud
> * Conta criada no AWS

Ao criar a conta e fazer o login, vemos a tela inicial do Docker Cloud no Swarm Mode, mas não vamos trabalhar nesse modo, que é para quando queremos integrar diversos hosts do Docker, para conversar entre si, e não é isso que queremos, queremos somente um para subir os nossos containers.

No menu lateral da esquerda, em *Repositories*, vemos as imagens salvas no nosso *Docker Hub*. Ainda no menu, vemos palavras já conhecidas, como *Containers* e *Services*, que fizemos na aula anterior. Ainda nesta aula, veremos sobre *Stacks*.

Além disso, temos também *Nodes* e *Node Clusters*. *Nodes* representa o *Docker Host*, ou seja, onde iremos rodar o Docker. No nosso caso, queremos rodar o Docker em Linux, que estará em uma máquina virtual da AWS. Ou seja, o node nada mais é do que um nó que estará gerenciado pela Amazon.

Para utilizarmos a infraestrutura que a Amazon nos provê dentro do Docker Cloud, precisamos fazer o link da conta da AWS com a conta do Docker Cloud. Fazemos isso também no menu lateral da esquerda, em *Cloud Settings*, na sessão *Cloud providers*, que lista quem podemos conectar à nossa conta do *Docker Cloud*.

Na AWS:
> * criar policy (Own Policy)
> * criar role (Role for cross-account access)

No DockerCloud:
> * criar um node
> * configurar node com os dados AWS

###Node Clusters###
Quando criamos um node, um nó, na verdade é criado por debaixo dos panos um node cluster, que é como se fosse uma coleção de vários nodes, onde podemos ter diversas máquinas virtuais falando uma com a outra, ou vários nodes em uma mesma máquina, mas como estamos fazendo uma aplicação simples, que não precisa de nada muito pesado, criamos um node só, então no nosso cluster só deve ter um nó rodando.

