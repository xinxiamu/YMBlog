---
title: docker安装各种应用软件
date: 2018-07-19 15:49:39
categories: docker
tags: docker安装应用
---

本文记录在docker各种应用程序的安装以及使用……

docker仓库： [hub repo](https://hub.docker.com/)

## 安装nginx

参考：
- http://www.runoob.com/docker/docker-install-redis.html


## 安装redis

参考：
- http://www.runoob.com/docker/docker-install-redis.html



## 安装mysql-server

仓库：https://hub.docker.com/r/library/mysql/

#### 拉取镜像：

    docker pull mysql:8.0.13  # 8.0.13为标签tag
    
#### 使用mysql镜像

    docker run -p 3306:3306 --name mymysql -v $PWD/conf:/etc/mysql/conf.d -v $PWD/logs:/logs -v $PWD/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:8.0.13    

命令说明：

- -p 3306:3306：将容器的 3306 端口映射到主机的 3306 端口。
- -v -v $PWD/conf:/etc/mysql/conf.d：将主机当前目录下的 conf/my.cnf 挂载到容器的 /etc/mysql/my.cnf。看下面补充内容说明。
- -v $PWD/logs:/logs：将主机当前目录下的 logs 目录挂载到容器的 /logs。
- -v $PWD/data:/var/lib/mysql ：将主机当前目录下的data目录挂载到容器的 /var/lib/mysql 。
- -e MYSQL_ROOT_PASSWORD=123456：初始化 root 用户的密码。

```text
补充：     
MySQL默认配置文件是 /etc/mysql/my.cnf 文件。如果想要自定义配置，建议向 /etc/mysql/conf.d 目录中创建 .cnf 文件。新建的文件可以任意起名，只要保证后缀名是 cnf 即可。新建的文件中的配置项可以覆盖 /etc/mysql/my.cnf 中的配置项。     
具体操作：   
首先需要创建将要映射到容器中的目录以及.cnf文件，然后再创建容器

    # pwd
    /opt
    # mkdir -p docker_v/mysql/conf
    # cd docker_v/mysql/conf
    # touch my.cnf
    # docker run -p 3306:3306 --name mysql -v /opt/docker_v/mysql/conf:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=123456 -d imageID
    4ec4f56455ea2d6d7251a05b7f308e314051fdad2c26bf3d0f27a9b0c0a71414
    
-p 3306:3306：将容器的3306端口映射到主机的3306端口
-v /opt/docker_v/mysql/conf:/etc/mysql/conf.d：将主机/opt/docker_v/mysql/conf目录挂载到容器的/etc/mysql/conf.d
-e MYSQL_ROOT_PASSWORD=123456：初始化root用户的密码
-d: 后台运行容器，并返回容器ID
imageID: mysql镜像ID
```
*不使用conf配置文件*:

参考：https://hub.docker.com/r/library/mysql/

创建容器mysql实例，端口为：3306

    docker run --name mysql-3306 -p 3306:3306 -v /server/mysql-3306/logs:/logs -v /server/mysql-3306/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:8.0.13 --port=3306  --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
    
查看配置参数列表：

    docker run -it --rm mysql:tag --verbose --help    

#### 查看容器启动情况

    [root@xr-server ~]# docker ps
    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                               NAMES
    415c0167fb5a        mysql:8.0.13        "docker-entrypoint.s…"   5 minutes ago       Up 5 minutes        0.0.0.0:3306->3306/tcp, 33060/tcp   mymysql

#### 进入mysql镜像

参考容器章节……

`docker exec -it 415c0167fb5a /bin/bash 或者 docker exec -it 415c0167fb5a bash`

`415c0167fb5a`为容器id

```text
[root@xr-server ~]# clear
[root@xr-server ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                               NAMES
415c0167fb5a        mysql:8.0.13        "docker-entrypoint.s…"   17 hours ago        Up 6 minutes        0.0.0.0:3306->3306/tcp, 33060/tcp   mymysql
[root@xr-server ~]# docker exec -it 415c0167fb5a /bin/bash 或者 docker exec -it 415c0167fb5a bash
root@415c0167fb5a:/# ls
bin   docker-entrypoint-initdb.d  home	 logs	opt   run   sys  var
boot  entrypoint.sh		  lib	 media	proc  sbin  tmp
dev   etc			  lib64  mnt	root  srv   usr
root@415c0167fb5a:/# 
```
    
## 安装zabbix

#### 启动一个mysql服务器实例

    docker run --name zabbix-mysql-server  \
    -e MYSQL_ROOT_PASSWORD="123456" \
    -e MYSQL_USER="zabbix" \
    -e MYSQL_PASSWORD="123456" \
    -e MYSQL_DATABASE="zabbix" \
    -p 3306:3306  \
    -v /server/dockers/zabbix/mysql/logs:/logs \
    -v /server/dockers/zabbix/mysql/data:/var/lib/mysql \
    -d mysql:8.0.13 \
    --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
       
#### 启动Zabbix server实例，并关联这个实例到已创建的MySQL服务器实例

    docker run  --name zabbix-server-mysql --hostname zabbix-server-mysql \
    --link zabbix-mysql-server:mysql \
    -e DB_SERVER_HOST="mysql" \
    -e MYSQL_USER="zabbix" \
    -e MYSQL_DATABASE="zabbix" \
    -e MYSQL_PASSWORD="123456" \
    -e MYSQL_ROOT_PASSWORD="123456" \
    -v /etc/localtime:/etc/localtime:ro \
    -v /server/dockers/zabbix/zabbix-server/alertscripts:/usr/lib/zabbix/alertscripts \
    -v /server/dockers/zabbix/zabbix-server/externalscripts:/usr/lib/zabbix/externalscripts \
    -p 10051:10051 \
    -p 10050:10050 \
    -d zabbix/zabbix-server-mysql:centos-4.0.2
    
    ------------------------------ 查看启动日志 ---------------
    docker logs zabbix-server-mysql
   

_注意_：必须加上属性` -e MYSQL_ROOT_PASSWORD="123456"`,否则mysql的`zabbix`用户没有操作mysql数据库的权限。    

>`--link`属性说明：   
docker run --link可以用来链接2个容器，使得源容器（被链接的容器）和接收容器（主动去链接的容器）之间可以互相通信，并且接收容器可以获取源容器的一些数据，如源容器的环境变量。
--link的格式：
--link <name or id>:alias
其中，name和id是源容器的name和id，alias是源容器在link下的别名。    

#### 启动Zabbix web 接口，并将它与MySQL服务器实例和Zabbix server实例关联

    docker run --name zabbix-web-nginx-mysql --hostname zabbix-web-nginx-mysql \
    --link zabbix-mysql-server:mysql \
    --link zabbix-server-mysql:zabbix-server \
    -e DB_SERVER_HOST="mysql" \
    -e MYSQL_ROOT_PASSWORD="123456" \
    -e MYSQL_USER="zabbix" \
    -e MYSQL_PASSWORD="123456" \
    -e MYSQL_DATABASE="zabbix" \
    -e ZBX_SERVER_HOST="zabbix-server" \
    -e PHP_TZ="Asia/Shanghai" \
    -p 8083:80 \
    -d zabbix/zabbix-web-nginx-mysql:centos-4.0.1
    
    ------------------------------ 查看启动日志 ---------------
    docker logs zabbix-web-nginx-mysql

```text
浏览器访问ip:8000查看
默认登录
username:Admin
password:zabbix
```
  
 _注意_ ：生产环境要做数据卷映射。以防止数据丢失。必须加上属性` -e MYSQL_ROOT_PASSWORD="123456"`,否则mysql的`zabbix`用户没有操作mysql数据库的权限。
    
####　启动zabbix-agent

    docker run --name zabbix-agent --link zabbix-server-mysql:zabbix-server -d zabbix/zabbix-agent:centos-4.0.2 
    
最后需要在web端将，zabbix-agent添加到zabbix-server的host列表里面。    

####  启动Zabbix Java gateway实例,并关联到zabbix-server  

用于监控jvm/tomcat性能。   
Zabbix监控Java应用程序的关键点在于：配置Zabbix-JavaGateway、让Zabbix-Server能够连接Zabbix-JavaGateway、Tomcat开启JVM远程监控功能等。

    docker run --name zabbix-java-gateway --link zabbix-server-mysql:zabbix-server -d zabbix/zabbix-java-gateway:latest