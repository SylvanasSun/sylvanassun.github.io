---
title:         Docker的那点事儿
date:        2017-11-19 18:00
author:     "Sylvanas Sun"
catalog:    true
categories: 
    - 后端
    - Docker
tags:
    - 后端
    - Docker
    - 2017
---



### Docker是什么？


----------



Docker是一个基于轻量级虚拟化技术的容器，整个项目基于Go语言开发，并采用了Apache 2.0协议。Docker可以将我们的应用程序打包封装到一个容器中，该容器包含了应用程序的代码、运行环境、依赖库、配置文件等必需的资源，通过容器就可以实现方便快速并且与平台解耦的自动化部署方式，无论你部署时的环境如何，容器中的应用程序都会运行在同一种环境下。

举个栗子，小明写了一个CMS系统，该系统的技术栈非常广，需要依赖于各种开源库和中间件。如果按照纯手动的部署方式，小明需要安装各种开源软件，还需要写好每个开源软件的配置文件。如果只是部署一次，这点时间开销还是可以接受的，但如果小明每隔几天就需要换个服务器去部署他的程序，那么这些繁琐的重复工作无疑是会令人发狂的。这时候，Docker的用处就派上场了，小明只需要根据应用程序的部署步骤编写一份Dockerfile文件（将安装、配置等操作交由Docker自动化处理），然后构建并发布他的镜像，这样，不管在什么机器上，小明都只需要拉取他需要的镜像，然后就可以直接部署运行了，这正是Docker的魅力所在。

那么镜像又是什么呢？镜像是Docker中的一个重要概念：

 - Image（镜像）：它类似于虚拟机中使用到的镜像，由于任何应用程序都需要有它自己的运行环境，Image就是用来提供所需运行环境的一个模板。

 - Container（容器）：Container是Docker提供的一个抽象层，它就像一个轻量级的沙盒，其中包含了一个极简的Linux系统环境与运行在其中的应用程序。Container是Image的运行实例（Image本身是只读的，Container启动时，Docker会在Image的上层创建一个可写层，任何在Container中的修改都不会影响到Image，如果想要在Image保存Container中的修改，Docker采用了基于Container生成新的Image层的策略），Docker引擎利用Container来操作并隔离每个应用（也就是说，每个容器中的应用都是互相独立的）。

其实从Docker与Container的英文单词原意中就可以体会出Docker的思想。Container可以释义为集装箱，集装箱是一个可以便于机械设备装卸的封装货物的通用标准规格，它的发明简化了物流运输的机械化过程，使其建立起了一套标准化的物流运输体系。而Docker的意思为码头工人，可以认为，Docker就像是在码头上辛勤工作的工人，把应用打包成一个个具有某种标准化规格的"集装箱"（其实这里指出的集装箱对应的是Image，在Docker中Container更像是一个运行中的沙盒），当货物运输到目的地后，码头工人们（Docker）就可以把集装箱拆开取出其中的货物（基于Image来创建Container并运行）。这种标准化与隔离性可以很方便地组合使用多个Image来构建你的应用环境（Docker也提倡每个Image都遵循单一职责原则，也就是只做好一件事），或者与其他人共享你的Image。

> 本文作者为[SylvanasSun(sylvanas.sun@gmail.com)][1]，首发于[SylvanasSun’s Blog][2]。
> 原文链接：https://sylvanassun.github.io/2017/11/19/2017-11-19-docker_introduction/
> （转载请务必保留本段声明，并且保留超链接。）


### Docker VS 虚拟机


----------



在上文中我们提到了Docker是基于轻量级虚拟化技术的，所以它与我们平常使用的虚拟机是不一样的。虚拟机技术可以分成以下两类：

![系统虚拟机](https://upload.wikimedia.org/wikipedia/commons/0/08/Hardware_Virtualization_%28copy%29.svg)

 - 系统虚拟机：通过软件对计算机系统的模拟来提供一个真实计算机的替代品。它是物理硬件的抽象并提供了运行完整操作系统所需的功能。虚拟机通过物理机器来管理和共享硬件，这样实现了多个虚拟机环境彼此之间的隔离，一台机器上可以运行多个虚拟机，每个虚拟机包括一个操作系统的完整副本。在系统虚拟机中，所运行的所有软件或操作都只会影响到该虚拟机的环境。我们经常使用的VMWare就是系统虚拟机的实现。

 - 程序虚拟机：允许程序独立运行在平台之外。比较典型的例子就是JVM，Java通过JVM这一抽象层使得Java程序与操作系统和硬件平台解耦（因为每个Java程序都是运行在JVM中的），因此实现了所谓的compile once, run everywhere。

Docker所用到的技术与上述两种都不相同，它使用了更轻量级的虚拟化技术，多个Container共享了同一个操作系统内核，并且就像运行在本地上一样。Container技术相对于虚拟机来说，只是一个应用程序层的抽象，它将代码与依赖关系打包到一起，多个Container可以在同一台机器上运行（意味着一个虚拟机上也可以运行多个Container），并与其它Container共享操作系统内核，每一个Container都在用户空间中作为一个独立的进程运行，这些特性都证明了Container要比虚拟机更加灵活与轻量（一般都是结合虚拟机与Docker一起使用）。

![](http://wx3.sinaimg.cn/large/63503acbly1flnj7b3v1lj214x0d7mxs.jpg)

Container技术其实并不是个新鲜事物，最早可以追溯到UNIX中的chroot（在1979年的V7 Unix中引入），它可以改变当前正在运行的进程及其子目录的根目录，在这种修改过的环境下运行的程序不能在指定的目录树之外访问文件，从而限制用户的活动范围，为进程提供了隔离空间。

之后各种Unix版本涌现出很多Container技术，在2006年，Google提出了"Process Containers"期望在Linux内核中实现进程资源隔离的相关特性，由于Container在Linux内核中的定义过于宽泛混乱，后来该项目改名为CGroups（Control Groups），实现了对进程的资源限制。

2008年，LXC（Linux Containers）发布，它是一种在操作系统层级上的虚拟化方法，用于在Linux系统上通过共享一个内核来运行多个互相隔离的程序（Container）。LXC正是结合了Linux内核中的CGroups和对分离的名称空间的支持来为应用程序提供了一个隔离的环境。而Docker也是基于LXC实现的（Docker的前身是dotClound公司中的内部项目，它是一家提供PaaS服务的公司。），并作出了许多改进。

### 使用Docker


----------



在使用Docker之前你需要先安装Docker（这好像是一句废话。。。），根据不同的平台安装方法都不相同，可以去参考[Install Docker | Docker Documentation][3]或者自行Google。

安装完毕之后，输入`docker --version`来确认是否安装成功。

```shell
$ docker --version
Docker version 17.05.0-ce-rc1, build 2878a85
```

从[Docker Hub][4]中可以pull到其他人发布的Image，我们也可以注册一个账号去发布自己的Image与他人共享。

```shell
[root@Jack ~]# docker search redis # 查看redis镜像是否存在
[root@Jack ~]# docker pull redis # 拉取redis镜像到本机
Using default tag: latest
Trying to pull repository docker.io/library/redis ... 
latest: Pulling from docker.io/library/redis
Digest: sha256:cd277716dbff2c0211c8366687d275d2b53112fecbf9d6c86e9853edb0900956
[root@Jack ~]# docker images # 查看镜像信息
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
docker.io/python    3.6-onbuild         7195f9298ffb        2 weeks ago         691.1 MB
docker.io/mongo     latest              d22888af0ce0        2 weeks ago         360.9 MB
docker.io/redis     latest              8f2e175b3bd1        2 weeks ago         106.6 MB
```

有了Image，之后就可以在其之上运行一个Container了，命令如下。

```shell
[root@Jack ~]# docker run -d -p 6379:6379 redis # 运行redis，-p代表将本机上6379端口映射到Container的6379端口 -d代表在后台启动
[root@Jack ~]# docker ps -a # 查看容器信息，如果不加-a只会显示当前运行中的容器
# 如果想要进入容器中，那么需要执行以下命令
[root@Jack ~]# docker ps # 先获得容器的id
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
1f928073b7eb        redis               "docker-entrypoint.sh"   45 seconds ago      Up 44 seconds       0.0.0.0:6379->6379/tcp   desperate_khorana
[root@Jack ~]# docker exec -it 1f928073b7eb /bin/bash # 然后再执行该命令进入到容器中
root@1f928073b7eb:/data# touch hello_docker.txt # 在容器中创建一个文件
root@1f928073b7eb:/data# exit # 退出
exit
[root@Jack ~]# 
# 也可以在启动时直接进入 命令如下
[root@Jack ~]# docker run -d -it -p 6379:6379 redis /bin/bash
```

我们对Container做出了修改，如果想要保留这个修改，可以通过commit命令来生成一个新的Image。

```shell
# -m为描述信息 -a为作者 1f9是你要保存的容器id 取前3个字符 docker可以自行识别
# sylvanassun/redis为镜像名 :test 为一个tag 一般用于标识版本
[root@Jack ~]# docker commit -m "test" -a "SylvanasSun" 1f9 sylvanassun/redis:test
sha256:e7073e8e5bd70b8d58092fd6bd8c2551e65dd29241c235eddf2a7f4b4b25cbbd
[root@Jack ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
sylvanassun/redis   test                e7073e8e5bd7        2 seconds ago       106.6 MB
docker.io/python    3.6-onbuild         7195f9298ffb        2 weeks ago         691.1 MB
docker.io/mongo     latest              d22888af0ce0        2 weeks ago         360.9 MB
docker.io/redis     latest              8f2e175b3bd1        2 weeks ago         106.6 MB
```

想删除一个容器或镜像也很简单，但在删除镜像前需要先删除依赖于它的容器。

```shell
[root@Jack ~]# docker stop 1f9 # 关闭运行中的容器，相应的也有docker start id命令来启动一个容器
1f9
[root@Jack ~]# docker rm 1f9 # 删除容器
1f9
[root@Jack ~]# docker rmi e70 # 删除上面保存的镜像
Untagged: sylvanassun/redis:test
Deleted: sha256:e7073e8e5bd70b8d58092fd6bd8c2551e65dd29241c235eddf2a7f4b4b25cbbd
Deleted: sha256:751db4a870e5f703082b31c1614a19c86e0c967334a61f5d22b2511072aef56d
```

如果想要自己构建一个镜像，那么需要编写Dockerfile文件，该文件描述了镜像的依赖环境以及如何配置你的应用环境。

```dockerfile
# 使用python:2.7-slim 作为父镜像
FROM python:2.7-slim

# 跳转到/app 其实就是cd命令
WORKDIR /app

# 将当前目录的内容(.)复制到镜像的/app目录下
ADD . /app

# RUN代表运行的shell命令，下面这条命令是根据requirements.txt安装python应用的依赖包
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# 暴露80端口让外界访问
EXPOSE 80

# 定义环境变量
ENV NAME World

# 当容器启动时执行的命令，它与RUN不同，只在容器启动时执行一次
CMD ["python", "app.py"]
```

然后就可以通过`docker build -t xxx/xxxx .`命令来构建镜像，`-t`后面是镜像名与tag等信息，注意`.`表示在当前目录下寻找Dockerfile文件。

学会如何构建自己的镜像之后，你是否也想将它发布到[Docker Hub][4]上与他人分享呢？要想做到这一点，需要先注册一个[Docker Hub][4]账号，之后通过`docker login`命令登录，然后再`docker push image name`，就像在使用Git一样简单。

关于Docker的更多命令与使用方法，请参考[Docker Documentation | Docker Documentation][5]，另外我还推荐使用[Docker Compose][6]来构建镜像，它可以很方便地组合管理多个镜像。

### 结语


----------



Docker提供了非常强大的自动化部署方式与灵活性，对多个应用程序之间做到了解耦，提供了开发上的敏捷性、可控性以及可移植性。同时，Docker也在不断地帮助越来越多的企业实现了向云端迁移、向微服务转型以及向DevOps模式的实践。

如今，微服务与DevOps火爆程度日益渐高，你又有何理由选择拒绝Docker呢？让我们一起选择拥抱Docker，拥抱未来！


[1]: https://github.com/SylvanasSun
[2]: https://sylvanassun.github.io/
[3]: https://docs.docker.com/engine/installation/
[4]: https://hub.docker.com/
[5]: https://docs.docker.com/
[6]: https://docs.docker.com/compose/