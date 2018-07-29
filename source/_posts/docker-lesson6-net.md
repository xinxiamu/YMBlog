---
title: docker学习-第六课：使用网络
date: 2018-07-26 20:23:35
categories: docker
tags: docker网络
---

本节概述docker中与网络相关的一些操作，如docker与主机之间的网络互通，docker之间的网络互相访问等……

参考网址： https://docs.docker.com/network/

## 外部访问网络

容器中可以运行一些网络应用，要让外部也可以访问这些网络应用，可以使用`-P`或者`-p`参数来指定端口映射。  

当使用`-P`标记时，Docker会随机主机的一个49000~49900端口到容器的内部端口。   
使用`docker container ps`可以查看到。 

    docker run -d -P --name ngxin-net nginx

创建容器`nginx-net`并后台启动。 

       [vagrant@ymu ~]$ sudo docker run -d -P --name ngxin-net nginx
       cfb84ca5f6837d14ea5f9c4d670ca4f9b126fbbe3e0d78f9cbaa949772f892be
       [vagrant@ymu ~]$ sudo docker container ps
       CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                   NAMES
       cfb84ca5f683        nginx               "nginx -g 'daemon of…"   14 seconds ago      Up 13 seconds       0.0.0.0:32768->80/tcp   ngxin-net
       d120d176e25c        nginx               "nginx -g 'daemon of…"   13 days ago         Up 3 minutes        0.0.0.0:80->80/tcp      nginx

可以看到容器`ngxin-net`的端口映射信息，把主机32768端口映射到了容器的80端口。这样，在主机就可以访问：localhost:32768

同样，可以查看应用访问日志： 

    [vagrant@ymu ~]$ sudo docker logs -f ngxin-net

另外，小写`-p`可以明确指定主机和容器之间的端口映射。 并且,在一个指定端口上只可以绑定一个容器。      
支持的格式： 

- ip:hostPort:containerPort
- ip::containerPort 
- hostPort:containerPort

前面一个为主机端口，后面一个为容器端口。 

常用：

`-p 80:80`

将本地主机80端口映射到容器80端口。

## 容器互联

容器与容器之间的端口互通。 互相访问容器信息。 

使用`--link`参数可以让容器之间安全的进行交互。  

下面创建一个新的redis数据库容器：

    docker run --name redis-test -d  redis
    
然后创建一个web容器：

    docker run -d -p 80:80 --name nginx-web --link redis-test nginx 
    
此时，redis-test容器和nginx-web容器建立了互联关系。  

`--link` 参数格式 `--link name：alias`，其中name是容器名称，alias是这个连接的别名。 

 可以通过`docker ps`查看连接信息。   
 这样，nginx-web容器将可以访问redis-test容器中的信息。  
 
 Docker在两个容器之间建立起了一个安全隧道，而且不用映射它们的端口到宿主主机上。启动redis-test的时候，采用大写`-P`，从而避免了暴露数据库端口到外部网络上。  
 
 
        

