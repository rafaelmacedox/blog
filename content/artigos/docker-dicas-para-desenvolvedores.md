+++
title: "Docker Dicas Para Desenvolvedores"
date: 2022-05-25T01:24:47-03:00
author = "Rafael Macedo"
description = "Algumas dicas retiradas de anotações pessoais realizadas após um curso sobre Docker para Desenvolvedores"
tags = [
    "docker"
]
categories = [
    "dicas"
]
+++

Um bom tempo (quase 2 anos hahaha) trabalhei com Docker sem nunca ter simplesmente parado para realmente ver a documentação ou algum curso, e depois de tanto passar sufoco por simplesmente não saber realmente como tudo funciona, resolvi fazer um curso inteiro somente sobre Docker. E hoje, algo meio como um bloco de anotações, vou repassar para você apenas o que eu considero mais importante e útil para a nossa produtividade durante o dia a dia de trabalho como DESENVOLVEDOR. 

Importante ressaltar que é para DEVS, por que caso você seja da área de DevOps, obviamente você precisará ter mais embasamento sobre o que estará sendo dito aqui.

## Ambiente

Primeiramente é muito importante ressaltar que o Docker é basicamente um PROCESSO e foi criado utilizando os recursos do Linux, como os namespaces, cgroups e file system. 

- Namespaces são a forma de isolar processos de um sistema operacional, e basicamente um container é um processo isolado com diversos processos filhos.
- Cgroups servem para controlar os recursos operacionais do container, como memoria e cpu, de forma que não interfira nos demais recursos da sua máquina (Docker Host).

![Docker Cgroup and namespaces](/static/images/kernel-cgroups-namespaces.jpg)

- File System ou OFS (Overlay File System) ajuda o Docker a funcionar em camadas, onde é possível pegar apenas a diferença do que foi alterado e assim não precisando fazer diversas cópias INTEIRAS das coisas.

Então importante lembrar que caso você esteja no Windows e utilizando o Docker Desktop puramente, saiba que você não estará desenvolvendo com 100% da performance que poderia, pois basicamente o Docker Desktop esta virtualizando um SO Linux para utilizar como Docker Host. Por isso é muito bacana, caso esteja desenvolvendo no Windows, utilizar o WSL2 para desfrutar de todos os recursos e performance que o Docker oferece. (Em breve vou fazer um artigo sobre o WSL2 também)

## Docker Visão Geral

Em uma visão geral, o Docker é composto pelas seguintes partes:

![Docker Overview](/static/images/docker-overview.png)

- Docker Host (O seu computador em sí e onde encontra-se os containers)
- Client (O terminal do seu computador quem enviará os comandos)
- Registry (Repositório de imagens para Docker, por exemplo o DockerHub.com)

## Primeiros passos

A primeira coisa a entender é que todo container é criado a partir de uma IMAGEM, essa imagem pode ser um ubuntu, nginx, nodejs, postgresql ou qualquer outra coisa que você encontrar no repositório de imagens.

Use: `docker run nginx`
- Cria e inicia um container utilizando a imagem nginx e trava o terminal no processo

Use: `docker run -d nginx` (Dispatch)
- Cria, inicia um container utilizando a imagem nginx e libera o terminal

Use: `docker ps`
- Lista todos os containers RODANDO

** Interessante olharmos que o container está rodando, possuí um CONTAINER ID e um CONTAINER NAME (isso será útil posteriormente) **

Use `docker run --name webserver nginx`
- Cria, inicia um container utilizando o NGINX e o nomeia como 'webserver'

Use `docker stop webserver`
- Finaliza um container

Use `docker rm webserver`
- Excluí um container

Use: `docker ps -a`
- Lista todos os containers EXISTENTES

Use `docker run --rm nginx`
- Cria, inicia um container utilizando o nginx e o destrói assim que o processo for finalizado

Use `docker run -it ubuntu bash`
- Cria, inicia um container utilizando o UBUNTU e acesso o sheel do container.
- '-i (Modo interativo, STDIN ativo, para permitir nossa interação com o container )'
- '-t (TTY é simplesmente um terminal ao qual você está conectado. Uma interface da qual você pode dar comandos texto a maquina)'
- 'bash (Meio autoexplicativo, vai abrir o sheel Bash)

 ** Também é possível acessar o container como dito acima, porém em containers já ativos.
Use `docker exec -it nginx bash`

### Redes

O Docker funciona com diversos tipos de redes, sendo eles:
- bridge (Um container fala com o outro) (Padrão)
- host (Mescla o docker com o host)
- overlay (Não muito usado) (Se tiver vários dockers em máquinas diferentes)
- maclan (Sinceramente não lembro)
- none (Não tem rede, é só o container isolado)

Ter rede configurada é super importante para manter a comunicação correta entre os containers.

Use: `docker network ls`
- Lista todas as redes existentes e seu respectivo driver (tipo).

Use: `docker network create --driver bridge minharede`
- Cria uma rede com o nome 'minharede' e o tipo 'bridge', ou seja, todos os containers que estiverem nesta rede irão poder se comunicar.

Use: `docker run --network minharede nginx`
- Cria um container conectado na rede com o nome 'minharede'

Use `docker network connect minharede meucontainer`
- Conecta o container com o nome 'meucontainer' na rede com o nome 'minharede'

Use `docker network desconnect minharede meucontainer`
- Desconecta o container com o nome 'meucontainer' da rede com o nome 'minharede'

Use: `docker network rm minharede`
- Deleta a rede com o nome 'minharede'. Note que caso algum container estiver conectado nesta rede não será possível executar este comando. Sendo necessário desconectar o container da rede ou deletá-lo.

Beleza, mas comigo sempre foi comum acontecer algo assim por exemplo: Eu iniciava um container postgresql `docker run -d postgresql` que por padrão utiliza a porta '5432' por padrão, porém ao tentar conectar em um software como o Dbeaver, simplesmente eu não conseguia conectar em 'localhost:5432'. 

Mas porque? Lembra que acabamos de falar que por padrão o docker cria containers na rede tipo 'bridge', ou seja, que não compartilha portas com o docker host (sua máquina). Então, poderíamos criar o container em uma rede do tipo 'host', que comunica diretamente com nossa maquina. Porém isso pode não ser a melhor opção em alguns casos por que caso você precise conectar outro container no seu postgresql esse container não enxergaria a porta por estar em outra rede.

Por isso possuímos a opção de expor uma porta diretamente ao criar o container.

Use: `docker run -p 8080:80 nginx`
- Basicamente estamos dizendo que queremos expor a porta 80 DO NGINX na porta 8080 da minha MAQUINA.

Beleza! E agora se precisarmos acessar de DENTRO DO CONTAINER um serviço que está DENTRO DA NOSSA MÁQUINA, como por exemplo, um container NODEJS conectar em um MYSQL que está rodando em nosso windows? Simples! No nodejs basta usar o 'http://host.docker.internal:3306' (no caso o 3306 é a porta padrão do mysql)

### Bind Mounts

Vale relembrar que cada container é baseado em uma imagem IMUTÁVEL. Ou seja, não modificamos ela, então todas as vezes que o container é finalizado tudo que fizemos dentro dele é perdido, e pra que isso não aconteça temos a opção de interligar uma pasta do Docker Host no container.

Use: `docker run -v source/:/usr/share/nginx/html nginx`
- Basicamente estamos fazendo um bind da pasta local chamada 'SOURCE' para dentro do container na pasta 'HTML' do nginx. 

Com isso tudo que for alterado na pasta 'html' dentro do container estará também na pasta 'source' da nossa máquina. 
**Mesmo que a pasta source ou a pasta html não existam, o docker irá criá-las.**


### Volumes

Caso seja necessário utilizar uma MESMA PASTA do Docker Host em DIVERSOS containers, podemos criar VOLUMES e utilizá-los em qualquer container.

Use: `docker volume ls`
- Lista todos os volumes já criados

Use: `docker volume create MEUVOLUME`
- Irá criar um novo volume

Use: `docker run --mount type=VOLUME,source=MEUVOLUME,target=/usr/share/nginx/html nginx`
- Irá mapear o volume chamado MEUVOLUME para dentro da pasta HTML do container nginx, e podemos subir quantos containers nginx quisermos utilizando exatamente o mesmo conteúdo.

**Importante para Windows**
Os volumes criados com o 'wsl2' vão ficar dentro do seguinte caminho:
'\\wsl$\docker-desktop-data\version-pack-data\community\docker\volumes\'

## Imagens e Registries

As imagens vem do DockerHub por padrão, mas as empresas podem ter seus próprios images registry server, como por exemplo o Nexus.

Use: `docker pull ubuntu`
- Para somente baixar uma imagem do ubuntu do docker, mas sem subir um container.

Use: `docker images`
- Lista todas as imagens já baixadas.

Use: `docker rmi ubuntu`
- Irá excluir a imagem do Docker Host (sua máquina)

# Conclusão

Então vou parar por aqui por que creio que já temos dicas o suficiente para um DEV utilizar Docker no dia a dia sem grandes problemas.

O assunto de imagens Docker é muito mais extenso que imaginamos, com as possibilidades de criarmos nossas próprias imagens com o DockerFile, Docker Composes, utilização em esteiras de CI/CD e diversas outras utilidades, o assunto imagens necessita de um artigo apenas sobre isso. O qual pretendo fazer em breve.

Um grande abraço e até a próxima!

