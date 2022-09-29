---
title: docker常用命令收藏
date: 2019-01-03 14:31:42
categories: docker
tags:
---

### 容器

#### 更改已启动容器属性

比如，如果在启动容器时候忘记设定随docker服务启动而自启动，那么可以执行以下命令更改属性：

更改容器`jenkins`启动属性：
```shell
docker update --restart=always jenkins # 设置自启动
docker update --restart=no jenkins # 取消自启动
```

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
    
#### 查看容器运行ip地址

下面查看容器名为`rabbitmq`的容器的ip地址：

```shell script
docker inspect --format '{{.NetworkSettings.IPAddress}}' rabbitmq
```    

或者
```shell script
docker inspect rabbitmq
```
然后查找`IPAddress`
    
### 镜像

#### 删除所有镜像

    docker rmi `docker images -q`
    
#### 按条件删除镜像

1.没有打标签     

    docker rmi `docker images -q | awk '/^<none>/ { print $3 }'`
    
2.镜像名包含关键字

    docker rmi --force `docker images | grep doss-api | awk '{print $3}'`    //其中doss-api为关键字 
    
3.删除所有none镜像

    docker rmi `docker images | grep  "<none>" | awk '{print $3}'`    
    
#### 导出导入镜像

_导出镜像：_

如果要存出镜像到本地文件，可以使用docker save命令。例如：

```shell script
[root@xr-server-dev images]# docker save -o eureka-server.tar 192.168.0.3:8082/xrlj/eureka-server:0.0.1
```

如上，把镜像导出当前文件目录下，名字`eureka-server.tar`。

_导入镜像:_

可以使用docker load从存出的本地文件中再导入到本地镜像库。例如：

```shell script
[root@iZj6c37qyt7ur3kr6b8u5nZ docker-images]# docker load -i eureka-server.tar
```

如上，在当前目录下，导入镜像`eureka-server.tar`。


#### 临时运行镜像的一个实例

```shell script
docker run -rm -p 1111:1111  192.168.0.3:8082/xrlj/eureka-server:0.0.1
```

关闭运行后，会自动删除，不会创建容器。

#### 查看容器id，判断容器是否存在

```shell
docker inspect --format="{{.Id}}" 容器名称
```

容器存在，输出容器id。不存在，则报错
                
