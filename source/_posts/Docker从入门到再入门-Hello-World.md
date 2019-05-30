---
title: Docker从入门到再入门 - Hello World (二)
author: zranbo
abbrlink: 83fde42c
tags:
  - Docker
categories: []
date: 2019-05-30 15:39:00
---
** Hello World **

通过一个简单的 `hello world` 的例子体验一下 Docker 的使用

`docker search` 可以找到我们想要的镜像
```bash
 $ docker search hello-world
NAME                                       DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
hello-world                                Hello World! (an example of minimal Dockeriz…   936                 [OK]                
kitematic/hello-world-nginx                A light-weight nginx container that demonstr…   124                                     
tutum/hello-world                          Image to test docker deployments. Has Apache…   60                                      [OK]
dockercloud/hello-world                    Hello World!                                    15                                      [OK]
crccheck/hello-world                       Hello World web server in under 2.5 MB          7                                       [OK]
ppc64le/hello-world                        Hello World! (an example of minimal Dockeriz…   2                                       
souravpatnaik/hello-world-go               hello-world in Golang                           1                                       
carinamarina/hello-world-app               This is a sample Python web application, run…   1                                       [OK]
markmnei/hello-world-java-docker           Hello-World-Java-docker                         1                                       [OK]
ansibleplaybookbundle/hello-world-db-apb   An APB which deploys a sample Hello World! a…   0                                       [OK]
koudaiii/hello-world                                                                       0                                       
kevindockercompany/hello-world                                                             0                                       
ansibleplaybookbundle/hello-world-apb      An APB which deploys a sample Hello World! a…   0                                       [OK]
burdz/hello-world-k8s                      To provide a simple webserver that can have …   0                                       [OK]
uniplaces/hello-world                                                                      0                                       
ansibleplaybookbundle/hello-world          Simple containerized application that tests …   0                                       
nirmata/hello-world                                                                        0                                       [OK]
infrastructureascode/hello-world           A tiny "Hello World" web server with a healt…   0                                       [OK]
stumacsolutions/hello-world-container                                                      0                                       
mbrainar/hello-world                       Python-based hello-world web service            0                                       
sharor/hello-world                                                                         0                                       
s390x/hello-world                          Hello World! (an example of minimal Dockeriz…   0                                       
kousik93/hello-world                                                                       0                                       
ebenjaminv9/hello-world                    Hello-world                                     0                                       
jensendw/hello-world                                                                       0                                       
```
通过 `docker pull`命令可以把我们想要的镜像下载下来。
```bash
$ docker pull hello-world
Using default tag: latest
latest: Pulling from library/hello-world
1b930d010525: Pull complete 
Digest: sha256:0e11c388b664df8a27a901dce21eb89f11d8292f7fca1b3e3c4321bf7897bffe
Status: Downloaded newer image for hello-world:latest
```
`docker images` 可以查看我们下载下来的所有镜像
```bash
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              fce289e99eb9        4 months ago        1.84kB
```
 - `REPOSITORY` 是镜像的名称
 - `TAG` 镜像的标签，可以理解为镜像的版本
 - `IMAGE ID` 镜像的ID 一般唯一表示一个镜像可只使用前6位
 - `CREATED` 镜像的最后一次构建时间
 - `SIZE` 镜像的大小
 
从上一篇中我们知道，镜像(Image)和容器(Container)的关系就好像类和实例一样，因此如果想运行这个镜像，就必须先‘实例化’一下这个镜像。而这个‘实例’的过程就是 `docker run` 命令来完成。

```bash
$ docker run hello-world

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
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

我们需要注意的是，`docker run` 命令如果是一个本地不存在的镜像首先会尝试下载，然后运行容器。

`hello-world` 镜像再运行后会输出以上内容，输出以后容器自动停止，停止的容器并不会被删除。我们可以用 `docker ps -a` 命令查看所有的容器。

注意：查看容器的命令 `docker ps` 但此时只能查看当前正在运行的容器，而想要查看所有容器(包含未运行)就需要 `-a` 参数

```bash
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
814d3d9a317b        hello-world         "/hello"            3 minutes ago       Exited (0) 3 minutes ago                       boring_ishizaka
```
当我们不需要一个容器的时候，可以删除。

```bash
$ docker rm 4619207cb6a3
4619207cb6a3
```
删除一个容器用到的 `rm` 后面不一定必须跟 `CONTAINER ID`，而是只要能唯一表示一个容器的就可以，所以因为可以使用 `NAMES`

```bash
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS                          PORTS               NAMES
c1b20afd08bb        hello-world         "/hello"            About a minute ago   Exited (0) About a minute ago                       condescending_hopper
$ docker rm condescending_hopper
condescending_hopper
```
相同的一个镜像也可以删除。

```bash
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              fce289e99eb9        4 months ago        1.84kB
$ docker rmi fce289e99eb9
```
和删除容器类似，`rmi` 后面不一定是 `IMAGE ID` 只要是能唯一表示一个镜像的名称就可以，所以用`REPOSITORY`也可以。

但是删除容器需要注意，容器不能是在运行中，删除镜像要求不能有这个镜像的容器。
