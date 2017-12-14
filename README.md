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