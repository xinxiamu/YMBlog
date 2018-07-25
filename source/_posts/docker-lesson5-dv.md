---
title: docker学习-第五课：数据管理
date: 2018-07-19 16:25:30
categories: docker
tags: docker数据管理
---

本文介绍docker中数据的管理，持久化。在docker中需要持久化应用程序产生的数据，或者需要多个容器之间共享数据。在容器中管理数据主要有两种方式： 

- 数据卷（Data volumes）
- 数据卷容器（Data volumes containers） 

在新版本中，推荐使用`docker volume`子命令来管理`Docker`数据卷。   

## 数据卷

数据卷是一个特殊的目录，它将主机目录直接映射到容器，可供一个或者多个容器使用。它绕过`UFS`可以提供很多特性。 

- 数据卷在容器启动时初始化，如果容器使用的镜像在挂载点包含了数据，这些数据会拷贝到新初始化的数据卷中。
- 数据卷可以在多个容器之中共享和重用。 
- 对数据卷的修改会立马生效。
- 对数据卷的更新不会影响镜像。
- 数据卷默认会一直存在，即使容器被删除。 

_注_ : 使用docker中的数据卷，类似于系统使用`mount`挂载一个文件系统。 

### 创建数据卷

1. 使用`-v`创建一个数据卷挂载到容器中

`docker run --name nginx-data -v /mydir nginx`

执行如下命令即可查看容器构造的详情  
`docker inspect 容器ID`  

注： 也可以在`Dockerfile`中添加一个或者多个卷到由该镜像创建的任何容器中。

### 删除数据卷

数据卷是设计来做持久化的，它的生命周期独立于容器，Docker不会在容器删除后删除数据卷。所以，要手动明确删除某个数据卷。

删除容器的时候同时想删除数据卷可以使用下面命令：

    docker rm -v 容器ID
    
### 挂载宿主机目录作为数据卷

同样，使用`-v`可以将宿主机目录挂载到容器中，如下：

    docker run --name nginx-data2 -v /host-dir:/container-dir nginx

上面命令，会将宿主机目录`/host-dir`挂载到容器的`/container-dir`目录。 

- 本地目录的路径必须是绝对路径。  
- 如果宿主机路径不存在，Docker会自动创建。 

_注意_ : Dockerfile不支持这种形式。因为Dockerfile是用来传播和分享的，不同操作系统，路径表示不一样，所以目前暂时不支持。 

Docker挂载的数据卷默认权限是可读写，如果要改为只读，可以通过`：ro`指定。 

    docker run --name nginx-data2 -v /host-dir:/container-dir：ro nginx

加了`：ro`之后，就挂载伟只读了。 这样，在容器就只能读取`/container-dir`目录中的文件，不能写。

### 查看数据卷具体信息

可以通过下面命令查看：

    docker inspect web
    
从输出内容中可以看到数据卷相关的部分。 

## 数据卷容器     

如果有数据需要不断更新并在多个容器之间共享，那么你就需要建立数据卷容器了。

数据卷容器首先是个正常的容器，专门提供数据卷供其它容器挂载的。  

创建数据卷容器： 
    
    docker run --name nginx-volume -v /data nginx
    
然后，就可以在其它容器中使用`-volumes-from`来挂载nginx-volume容器中的数据卷。   

    docker run --name v1 --volumes-from nginx-volume nginx
    docker run --name v2 --volumes-from nginx-volume nginx

这样：  

- v1、v2便可以共享nginx-volume容器中的数据卷。  
- 即使nginx-volume容器停止，也不会有任何影响。   

也可以使用超过一个的`-volumes-from`参数来指定从多个容器挂载不同的数据卷。也可以从其它挂载了数据卷的容器来级联挂载数据卷。 

     docker run --name v3 --volumes-from v1 nginx
     
## 利用数据卷容器来备份、恢复、迁移数据卷 

### 备份

首先创建一个挂载容器卷`nginx-volume`的容器，并从主机挂载当前目录到容器`/backup`目录。如下： 

    docker run --name data-backup --volume-from nginx-volume -v  $(pwd):/backup centos/7 tar cvf /backup/backup.tar /nginx-volume
    
容器启动后，使用命令`tar`将容器卷`nginx-volume`备份为容器中/backup/backup.tar文件，也就主机当前目录下的名为backup.tar的文件。 

### 恢复

如果要恢复数据到一个容器，首先创建一个挂载空数据卷的容器`nginx-volume2`。 

    docker run -v /dbdata --name nginx-volume2 ngxin /bin/bash
    
然后创建另一个容器,挂载nginx-volume2容器卷中的数据卷,并使用`untar`解压备份文件到挂载的容器卷中。 

    docker run --vulume-from /nginx-volume2 -v $(pwd):/backup nginx untar xvf /backup/backup.tar
    
为了查看/验证恢复的数据,可以再启动一个容器挂载同样的容器卷来查看  

    dcoker run --volume-from nginx-volume2 nginx /bin/ls /dbdata    
