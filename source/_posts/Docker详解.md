---
title: Docker详解
toc: true
date: 2022-04-26 20:51:32
tags: docker
categories: 杂项
---

​​点击阅读更多查看文章内容<!--more-->

# 什么是Docker
在认识Docker之前，我们要先学习一下什么是**容器**？

## 什么是容器？
不知道大家有没有遇到过，有的时候同样的代码在你的主机上能运行，但是移动到另一台主机上就会报错，究其原因是因为两台主机的运行环境不同。那么我们有没有一种方法，可以在移动代码的时候连同代码的运行环境也一起移动，这样在运行代码的时候就不需要考虑运行环境的问题了。

这种方法就是——容器技术，简单地说，一个容器包含了完整的运行时环境：除了应用程序本身之外，这个应用所需的全部依赖、类库、其他二进制文件、配置文件等，都统一被打入了一个称为容器镜像的包中。通过将应用程序本身和其依赖容器化，操作系统发行版本和其他基础环境造成的差异，都被抽象掉了。

容器在宿主机操作系统中，在用户空间以分离的进程运行。容器技术是实现操作系统虚拟化的一种途径，可以让您在**资源受到隔离的进程中运行应用程序及其依赖关系**。

>**容器实际上就是一种资源隔离的进程**

**注意容器和虚拟化不同**
**虚拟化使得许多操作系统可同时在单个系统上运行。
容器则可共享同一个操作系统内核，将应用进程与系统其他部分隔离开。**
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/46a6ff09db98af3bcf3c318f66e46fd1_1740930666930.png)

**说完了容器我们就该回到Docker了**
Docker 属于 Linux 容器的一种封装，提供简单易用的容器使用接口。它是目前最流行的 Linux 容器解决方案。
简单来讲， Linux 容器不是模拟一个完整的操作系统，而是对进程进行隔离，相当于是在正常进程的外面套了一个保护层。对于容器里面的进程来说，它接触到的各种资源都是虚拟的，从而实现与底层系统的隔离。
Docker 将应用程序与该程序的依赖，打包在一个文件里面。运行这个文件，就会生成一个虚拟容器。程序在这个虚拟容器里运行，就好像在真实的物理机上运行一样。有了 Docker ，就不用担心环境问题。
总体来说， Docker 的接口相当简单，用户可以方便地创建和使用容器，把自己的应用放入容器。容器还可以进行版本管理、复制、分享、修改，就像管理普通的代码一样。

# Docker的架构
**Docker的三个基本概念**
- Image（镜像）：镜像（Image）可以看作是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。镜像不包含任何动态数据，其内容在构建之后也不会被改变。
- Container（容器）：镜像（Image）和容器（Container）的关系，就像是面向对象程序设计中的类和实例一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。
- Repository（仓库）：仓库可看成一个代码控制中心，用来保存镜像。


![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/773555f38af5c415c811492c92532ede_1740930666930.png)
|概念|说明|
|--|--|
|Docker 镜像(Image)|用来创建Docker容器的模板|
|Docker 容器(Container)|镜像运行的实体|
|Docker 客户端(Client)| 通过命令行或其它工具与Docker的守护进程(daemon)通信|
|Docker 主机(Host)| 一个物理或实体机器用于执行Docker的守护进程和容器|
|Docker Registry| 用来保存Docker镜像，可以理解为代码控制中的代码仓库，一个 Docker Registry 中可以包含多个仓库（Repository）；每个仓库可以包含多个标签（Tag）；每个标签对应一个镜像。通常，一个仓库会包含同一个软件不同版本的镜像，而标签就常用于对应该软件的各个版本。我们可以通过 <仓库名>:<标签> 的格式来指定具体是这个软件哪个版本的镜像。如果不给出标签，将以 latest 作为默认标签。 |
|Docker Machine| Docker Machine是一个简化Docker安装的命令行工具，通过一个简单的命令行即可在相应的平台上安装Docker，比如VirtualBox、 Digital Ocean、Microsoft Azure。|


# Docker容器内容
- 文件系统：
容器内有自己的文件系统，通常是镜像中的文件系统，容器内部的文件系统与宿主机是隔离的，但它会共享宿主机的内核。
- 运行的应用程序：
容器内部运行的就是你所指定的应用程序或服务。例如，可能是一个 Web 服务器、数据库、API 服务等。这些应用程序是根据容器镜像中定义的配置启动的。
- 环境变量：
容器通常会设置一些环境变量，这些环境变量可以在容器内部的应用程序中使用。例如，数据库的连接字符串、端口号、API 密钥等。
- 网络配置：
容器内有自己的网络接口，通常通过 Docker 的网络模式（如 bridge, host, overlay）与宿主机或其他容器通信。容器内部可以设置特定的 IP 地址和端口号。
容器与宿主机之间的通信是通过 Docker 的端口映射来完成的。
- 进程与 PID 空间：
容器内会启动应用程序进程，每个容器都有独立的进程空间，意味着容器内的进程与宿主机或其他容器的进程是隔离的。
容器内部的进程是从容器镜像启动的，并且它们的进程 ID（PID）与宿主机的 PID 不冲突。
- 日志与输出：
容器内的应用程序会产生日志和标准输出。这些日志通常会被 Docker 收集，并可以通过 docker logs 命令查看。
如果你没有显式地挂载日志文件到宿主机，日志仅存在于容器内部。
- 卷 (Volumes)：
如果容器需要持久化数据，Docker 卷可以用来将宿主机或外部存储挂载到容器内部。容器对数据卷的访问不会随着容器的删除而丢失。容器内的数据可以通过 挂载卷（如宿主机的文件夹或外部存储）来进行共享和持久化。

# Docker容器的使用
在启动docker容器后一般通过两种方式使用
1. 直接进入docker容器内部进行交互
`docker exec -it my-container bash`
2. 通过端口映射，在宿主机中访问容器的服务
`docker run -d -p 8080:80 --name my-container nginx`，假设在启动容器时配置了端口映射8080:80，这样可以通过访问宿主机的8080端口来使用docker内监听在80端口上的服务程序
# Docker常用命令
## 基础命令
- 启动Docker
 `systemctl start docker`
- 关闭Docker
`systemctl stop docker`
- 查看Docker状态
`systemctl status docker`

## 镜像命令
- 列出本机镜像
`docker images`
- 拉取镜像到本地
不加tag即拉取该镜像的最新版本latest，加tag则是拉取该镜像的指定版本，版本号可以在[docker hub](https://hub.docker.com/)中查看
`docker pull 镜像名`
`docker pull 镜像名:tag`
- 查找镜像
从docker hub中查找满足要求的镜像，然后可以从中选择合适的镜像下载
`docker search 镜像名`
- 运行镜像
`docker run 镜像名(:Tag)`
- 删除镜像
`docker image rm 镜像名`
-f 强制删除
- 设置镜像标签
`docker tag 镜像名/镜像ID 镜像名:TAG`

## 容器命令
- 查看正在运行的容器列表
`docker ps`
-a 查看所有容器（包括已停止的）
- 启动一个新的容器
`docker run -it 镜像名 /bin/bash`
-i:交互式操作
-t:终端
-d:指定容器的运行模式，默认不会进入容器
/bin/bash:放在镜像后的是命令，这里我们希望有个交互式的Shell，因此用的是/bin/bash
- 停止容器
`docker stop 容器名/容器ID`
- 启动已停止的容器
`docker start 容器名/容器ID `
- 删除容器
`docker rm -f 容器名/容器ID`
- 进入容器
`docker exec -it 容器名/容器ID /bin/bash`
- 退出终端
`exit`

# dockerfile
dockfile是一个用于制作docker镜像的源码文件，内容是构建镜像过程的指令，在一个文件夹中，如果有一个名字为Dockfile的文件，其内容满足语法要求，在这个文件夹路径下执行命令:docker build --tag name:tag .，就可以按照描述构建一个镜像了。name是镜像的名称，tag是镜像的版本或者是标签号，不写就是lastest。注意后面有一个空格和点。
[dockerfile的使用详解](https://blog.csdn.net/m0_46090675/article/details/121846718)

# docker-compose
Compose是用于定义和运行多容器Docker应用程序的工具。通过Compose，可以使用YAML文件来配置应用程序的服务。然后使用一个命令，就可以从配置中创建并启动所有服务。
Docker-Compose是一个容器编排工具，将所有的容器的部署方法、文件映射、容器端口映射等情况写在一个.yml或.yaml文件里，执行docker-compose up命令就像执行脚本一样，一个一个的安装并部署容器。

>**docker-compose将所管理的容器分为三层：**
工程（project）：docker compose运行目录下的所有yml文件
服务（service）：一个工程包含多个服务，每个服务中定义了容器运行的镜像、参数、依赖。
容器（container）：一个服务可包括多个容器实例，所有的容器都通过services来定义


**示例：**
```xml
version: "3"                                  //指定语法的格式的版本
services:                                     //定义服务
  nginx:                                       //服务的名称
    container_name: web-nginx    //容器的名称
    image: nginx:latest                   //所使用的镜像
    restart: always                         //随docker服务的启动而启动
    ports:
      - 80:80                                 //映射的端口
    volumes:
      - /root/compose/webserver:/usr/share/nginx/html  //本地与容器挂载的目录
```

[docker-compose详解](https://blog.51cto.com/u_14306186/2517474)
[常用服务配置参考](https://blog.csdn.net/pushiqiang/article/details/78682323?)
