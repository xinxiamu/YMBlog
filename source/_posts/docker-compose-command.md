---
title: Docker Compose常用命令
date: 2018-08-01 20:33:33
categories: docker
tags:
---

本节讨论Compose的一些常用命令。

## Compose命令说明

执行`docker-compose [COMMAND] --help` 查看具体某个命令的使用说明。

选项: 
- --verbose 输出更多调试信息。   
- --version 打印版本并退出。
- -f, --file FILE 使用特定的 compose 模板文件，默认为 docker-compose.yml 。
- -p, --project-name NAME 指定项目名称，默认使用目录名称。

## 命令

### build

用于构建或重新构建服务。

服务一旦构建后，将会带上一个标记名，例如 web_db。

可以随时在项目目录下运行 docker-compose build 来重新构建服务。
        
### help

获得一个命令的帮助。`docker compose --help`

### kill

通过发送 SIGKILL 信号来强制停止服务容器。支持通过参数来指定发送的信号，例如

    $ docker-compose kill -s SIGINT
    
### logs

查看服务的输出。`docker-compose logs`

### port

打印绑定的公共端口。

### ps

列出所有容器。

进入compose工程目录，执行下面命令：

    [root@izwz9guplfsq8ltfk0wnjiz ELK]# docker-compose ps
           Name                      Command               State                                Ports                              
    -------------------------------------------------------------------------------------------------------------------------------
    elk_elasticsearch_1   /usr/local/bin/docker-entr ...   Up      0.0.0.0:9200->9200/tcp, 0.0.0.0:9300->9300/tcp                  
    elk_kibana_1          /usr/local/bin/dumb-init - ...   Up      0.0.0.0:5601->5601/tcp                                          
    elk_logstash_1        /usr/local/bin/docker-entr ...   Up      0.0.0.0:5000->5000/tcp, 5044/tcp, 0.0.0.0:8088->8088/tcp,       
                                                                   9600/tcp            
### pull

拉取服务镜像。

### rm

删除停止的服务容器。

### run

在一个服务上执行一个命令。`docker-compose run ubuntu ping docker.com`

将会启动一个 ubuntu 服务，执行 ping docker.com 命令。    

如果不希望自动启动关联的容器，可以使用 --no-deps 选项，例如

    $ docker-compose run --no-deps web python manage.py shell
    
将不会启动 web 容器所关联的其它容器。  

说明：

- 给定命令将会覆盖原有的自动运行命令；
- 不会自动创建端口，以避免冲突。

### scale

设置同一个服务运行的容器个数。    

    $ docker-compose scale web=2 worker=3
    
###  start

启动一个已经存在的服务容器。

### stop

停止一个已经运行的容器，但不删除它。通过 docker-compose start 可以再次启动这些容器。

### up

构建，（重新）创建，启动，链接一个服务相关的容器。

定义的服务都将会按照顺序启动，除非它们已经在运行。

默认情况下，`docker-compose up`将会整合所有容器的输出，前台运行，退出的时候，所有容器将停止运行。

如果使用`docker-compose up -d`,将会在后台启动并运行所有的程序。

>默认情况下，如果该服务的容器已经存在，`docker-compose up`将会停止所有容器并尝试重新创建容器（保持使用volumes-from挂载的卷），以保证`docker-compose.yml`的修改生效。如果你不想容器被停止并重新创建，那么可以使用`docker-compose up --no-recreate`,这样将会启动已经停止的容器。

                                                            
                                                                   
            
