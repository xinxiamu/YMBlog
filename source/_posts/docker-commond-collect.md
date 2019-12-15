---
title: docker常用命令收藏
date: 2019-01-03 14:31:42
categories: docker
tags:
---

## 容器

#### 查看容器端口映射

    [root@sqjr-client-demo-server1-hn zookeeper]# docker port zk-server1
    2181/tcp -> 0.0.0.0:2181
    2888/tcp -> 0.0.0.0:2888
    3888/tcp -> 0.0.0.0:3888
    
#### 强制删除运行中的容器

    docker rm -f 容器名称    
    
#### 拷贝本地文件到容器

    docker cp 本地文件路径 ID全称:容器路径  
    
#### 删除所有容器

    docker rm `docker ps -a -q`
    
    //运行中的也删除
    docker rm -f `docker ps -a -q`
    
#### 按条件筛选之后删除容器

    docker rm `docker ps -a | grep xxxxx | awk '{print $1}'`    
    
## 镜像

#### 删除所有镜像

    docker rmi `docker images -q`
    
#### 按条件删除镜像

1.没有打标签     

    docker rmi `docker images -q | awk '/^<none>/ { print $3 }'`
    
2.镜像名包含关键字

    docker rmi --force `docker images | grep doss-api | awk '{print $3}'`    //其中doss-api为关键字 
    
3.删除所有none镜像

    docker rmi `docker images | grep  "<none>" | awk '{print $3}'`                