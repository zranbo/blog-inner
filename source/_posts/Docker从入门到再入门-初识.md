---
title: Docker从入门到再入门 - 初识 (一)
author: zranbo
abbrlink: f79e33ca
tags:
  - Docker
categories: []
date: 2019-05-29 20:27:00
---
<b>什么是Docker?</b>

![](https://zranbo.oss-cn-beijing.aliyuncs.com/blog/docker_intr.jpg)

以上图片摘自 [https://www.docker.com/](https://www.docker.com/) 

`Docker` 是一个基于 `Go` 语言的开源容器引擎。`Docker` 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 `Linux` 机器上，也可以实现虚拟化。并且容器的使用开销极低，启动速度也非常快。

<b>安装 Docker</b>

`Docker` 有两个版本：社区版(Community Edition，缩写 CE)和企业版（Enterprise Edition，缩写 EE）一般个人安装 CE 版本

Docker CE 的安装参考官方文档：

 - [Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/)
 - [Mac](https://docs.docker.com/docker-for-mac/install/)
 - [Windows](https://docs.docker.com/docker-for-windows/install/)
 
安装完成后，可以通过 `docker --version` 验证版本

{% note danger %} 
Linux 安装完成后如果是非 `root` 用户输入 `docker` 会提示出错: `permission denied` 
解决办法是将当前用户加入 `docker` 用户组
`sudo usermod -a -G docker $USER` 并重新启动系统
利用 `groups` 查看 `docker` 是否在其中
{% endnote %}

<b>几个概念</b>
 
 1. ###### 镜像 (`Image`)
 
 首先操作系统分为内核和用户空间。对于 `Linux` 而言，内核启动后，会挂载 `root` 文件系统为其提供用户空间支 持。而 `Docker` 镜像（`Image`），就相当于是一个 `root` 文件系统。比如官方镜像 `ubuntu:18.04` 就包含了完整的一套 `Ubuntu 18.04` 最小系统的 `root` 文件系统。

 `Docker` 镜像是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。镜像不包含任何动态数据，其内容在构建之后也不会被改变。
 
 2. ###### 容器 (`Container`)
 
 镜像 (`Image`) 和容器 (`Container`) 的关系，就像是面向对象程序设计中的类(`class`)和实例(`Instance`) 一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。

 容器的实质是进程，但与直接在宿主执行的进程不同，容器进程运行于属于自己的独立的 命名空间。因此容器可以拥有自己的 `root` 文件系统、自己的网络配置、自己的进程空间，甚至自己的用户 `ID` 空间。容器内的进程是运行在一个隔离的环境里，使用起来，就好像是在一个独立于宿主的系统下操作一样。这种特性使得容器封装的应用比直接在宿主运行更加安全。也因为这种隔离的特性，很多人初学 `Docker` 时常常会混淆容器和虚拟机。

 前面讲过镜像使用的是分层存储，容器也是如此。每一个容器运行时，是以镜像为基础层，在其上创建一个当前容器的存储层，我们可以称这个为容器运行时读写而准备的存储层为 容器存储层。

 容器存储层的生存周期和容器一样，容器消亡时，容器存储层也随之消亡。因此，任何保存于容器存储层的信息都会随容器删除而丢失。

 按照 Docker 最佳实践的要求，容器不应该向其存储层内写入任何数据，容器存储层要保持无状态化。所有的文件写入操作，都应该使用 数据卷 (`Volume`)、或者绑定宿主目录，在这些位置的读写会跳过容器存储层，直接对宿主(或网络存储) 发生读写，其性能和稳定性更高。

 数据卷的生存周期独立于容器，容器消亡，数据卷不会消亡。因此，使用数据卷后，容器删除或者重新运行之后，数据却不会丢失。
 
 3. ###### 仓库(`Repository`)
 
 镜像构建完成后，可以很容易的在当前宿主机上运行，但是，如果需要在其它服务器上使用这个镜像，我们就需要一个集中的存储、分发镜像的服务，`Docker Registry` 就是这样的服务。

 一个 `Docker Registry` 中可以包含多个**仓库（`Repository`）**；每个仓库可以包含多个**标签（`Tag`）**；每个标签对应一个镜像。

 通常，一个仓库会包含同一个软件不同版本的镜像，而标签就常用于对应该软件的各个版本。我们可以通过 **<仓库名>:<标签>** 的格式来指定具体是这个软件哪个版本的镜像。如果不给出标签，将以 `latest` 作为默认标签。

 以 `Ubuntu` 镜像 为例，`ubuntu` 是仓库的名字，其内包含有不同的版本标签，如，16.04, 18.04。我们可以通过 `ubuntu:16.04`，或者 `ubuntu:18.04` 来具体指定所需哪个版本的镜像。如果忽略了标签，比如 `ubuntu`，那将视为 `ubuntu:latest`。

 仓库名经常以 **两段式路径** 形式出现，比如 `jwilder/nginx-proxy`，前者往往意味着 `Docker Registry` 多用户环境下的用户名，后者则往往是对应的软件名。但这并非绝对，取决于所使用的具体 `Docker Registry` 的软件或服务。