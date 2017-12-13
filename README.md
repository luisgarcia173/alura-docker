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