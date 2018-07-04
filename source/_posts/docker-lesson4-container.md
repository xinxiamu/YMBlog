---
title: docker学习-第四课：容器
date: 2018-06-30 15:24:01
categories: docker
tags: docker 容器
---

容器是`docker`的又一核心组件之一。  

简单理解：容器就是独立运行的一个或者一组应用,以及一整套的运行态环境（提供运行态环境以及系统环境）。  

## 启动容器：

启动容器有两种方式：

- 基于镜像创建一个容器并启动。
- 将在终止状态(stopped)的容器重新启动。 

新版本推荐使用子命令`	docker container` 。

### 新建并启动容器

使用命令：`docker run`

输出hello world并终止容器：

    $docker run ubuntu:14.04 /bin/echo 'Hello world'
    Hello world
    
启动一个bash终端,允许用户进行交互。

    $docker run -t -i ubuntu:14.04 /bin/bash
    root@af8bae53bdd3:/#
    
其中, `-t`选项让Docker分配一个伪终端(pseudo-tty)并绑定到容器的标准输入上,`-i`则让容器的标准输入保持打开。

`docker	run`启动容器流程：

- 检查本地是否存在指定的镜像，没有则从仓库拉取。
- 利用镜像创建一个容器并启动。
- 分配一个文件系统，并且在添加一层可读写。
- 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去。
- 从地址池配置一个ip地址给容器。
- 执行用户指定的应用程序。
- 执行完毕后容器终止。

### 启动已终止容器

命令`docker	start`直接将一个已经终止的容器启动。

容器的核心是，应用程序以及应用程序需要的资源。除此之外，并没有其它的资源。可以在伪终端中利用`ps`或`top`来查看进程信息。

    mutian@mutian-ThinkPad-T440p:~$ sudo docker run -t -i centos:latest /bin/bash
    [root@b80715a7913e /]# ps
      PID TTY          TIME CMD
        1 pts/0    00:00:00 bash
       15 pts/0    00:00:00 ps
    [root@b80715a7913e /]# 

可以看到，容器只启动了`bash`，资源利用率极高，非常的轻量。以至于都可以随便删除，添加容器。

### 容器管理`docker	container`

在`Docker 1.13+`版本，推荐使用`docker container`来管理容器。

    $ docker container run ubuntu:17.10 /bin/echo 'Hello world'
    $ docker container start

### 后台运行容器

前台运行：

    $docker	run ubuntu:14.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"
    hello	world
    hello	world
    hello	world
    hello	world
    
输出信息一直在宿主中输出。

后台运行，添加参数`-d`：  

    $docker run -d ubuntu:14.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"
    77b2dc01fe0f3f1265df143181e7b9af5e05279a884f4776ee75350ea9d8017a
    
_注意_: 容器是否能长久运行，与`docker run`指定的命令有关，与`-d`无关。

用命令`docker ps`查看容器信息。

用命令`docker logs [container ID	or NAMES]` 获取容器输出信息。

-注_: 后面版本使用：

    $	docker	container	run	-d
    $	docker	container	ls
    $	docker	container	logs   
    
## 终止容器

命令`docker stop`可以终止一个运行中的容器。

另外，当`Docker`中指定的应用程序终止时候，容器自动终止。 

命令`docker ps -a`可以查看终止状态的容器。

    mutian@mutian-ThinkPad-T440p:~$ sudo docker ps -a
    [sudo] password for mutian: 
    CONTAINER ID        IMAGE                       COMMAND                  CREATED             STATUS                    PORTS               NAMES
    b80715a7913e        centos:latest               "/bin/bash"              About an hour ago   Up About an hour                              confident_colden
    04288423a400        mysql/mysql-server:latest   "/entrypoint.sh mysq…"   6 weeks ago         Exited (0) 6 weeks ago                        mysql
    37741b9edbd1        nginx:v2                    "nginx -g 'daemon of…"   4 months ago        Exited (0) 4 months ago                       web5
    9e724b4f0f39        nginx:v2                    "nginx -g 'daemon of…"   4 months ago        Created                                       web4
    a9d4f7554e91        nginx:v3                    "nginx -g 'daemon of…"   4 months ago        Exited (0) 4 months ago                       web3
    e28e7f69cbc8        nginx:v2                    "nginx -g 'daemon of…"   4 months ago        Exited (0) 4 months ago                       web2
    c6074acec608        nginx                       "nginx -g 'daemon of…"   4 months ago        Exited (0) 4 months ago                       webserver
    95088bff626d        hello-world                 "/hello"                 4 months ago        Exited (0) 4 months ago                       stupefied_mcnulty

命令`docker start`可以重启终止状态下的容器。 
命令`docker restart`将会终止一个一个运行的容器后并重启。

_Doker 1.13+_:

    docker container ls
    docker container start
    docker container restart
    
    