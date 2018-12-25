---
title: docker安装各种常用开发应用软件
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

### 使用redis镜像

#### 不使用自定义配置文件：
 
 
    docker run -p 6379:6379 --name redis-6379 --restart=always -v /server/data/redis-6379/data:/data  -d redis:5.0.3 redis-server --appendonly yes --requirepass "a1234567"

```text
命令说明：
-p 6379:6379 : 将容器的6379端口映射到主机的6379端口
-v /server/data/redis-6379/data:/data : 将主机中/server/data/redis-6379目录下的data挂载到容器的/data
redis-server --appendonly yes : 在容器执行redis-server启动命令，并打开redis持久化配置 
--requirepass "a1234567" : 设置认证密码 
--restart=always ： 随docker启动而启动
```

通过客户端连接进入redis-ci：

    [root@sqjr-client-demo-server1-hn ~]# docker exec -it redis-6379 redis-cli -a a1234567
    Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
    127.0.0.1:6379> set username zmt

在宿主机查看redis进程：

    [root@sqjr-client-demo-server1-hn ~]# ps -ef|grep redis
    polkitd  29683 29651  0 14:27 ?        00:00:01 redis-server *:6379
    root     29935 29453  0 14:51 pts/1    00:00:00 grep --color=auto redis

只能本机127.0.0.1连接，不能远程连接。默认是保护模式，不允许远程连接。

#### 使用自定义配置文件（推荐）


     docker run --name redis-6380 -p 6380:6380 --restart=always -v /server/data/redis-6380/data:/data -v /server/data/redis-6380/conf/redis.conf:/usr/local/etc/redis/redis.conf -d redis redis-server  /usr/local/etc/redis/redis.conf --appendonly yes
     
redis.conf配置文件基本内容，自由添加：
    
    # 指定Redis监听端口，默认端口为6379
    # 如果指定0端口，表示Redis不监听TCP连接
    port 6380
    
    #设置密码
    requirepass a1234567
    
    #### 生产环境要开启保护模式
    # 绑定的主机地址
    # 你可以绑定单一接口，如果没有绑定，所有接口都会监听到来的连接
    # bind 127.0.0.1
    
    #protected-mode yes #关闭，不要保护模式，远程可连接
    
    # 当客户端闲置多长时间后关闭连接，如果指定为0，表示关闭该功能
    timeout 0
    
    #一定要打开，否则无法启动
    #daemonize yes
    
    # 当Redis以守护进程方式运行时，Redis默认会把pid写入/var/run/redis.pid文件，可以通过pidfile指定
    pidfile /var/run/redis.pid
    
    # 指定日志记录级别，Redis总共支持四个级别：debug、verbose、notice、warning，默认为verbose
    # debug (很多信息, 对开发／测试比较有用)
    # verbose (many rarely useful info, but not a mess like the debug level)
    # notice (moderately verbose, what you want in production probably)
    # warning (only very important / critical messages are logged)
    loglevel verbose
    
    # 日志记录方式，默认为标准输出，如果配置为redis为守护进程方式运行，而这里又配置为标准输出，则日志将会发送给/dev/null
    logfile stdout
    
    # 设置数据库的数量，默认数据库为0，可以使用select <dbid>命令在连接上指定数据库id
    # dbid是从0到‘databases’-1的数目
    databases 16
    
    ################################ SNAPSHOTTING  #################################
    # 指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合
    # Save the DB on disk:
    #
    #   save <seconds> <changes>
    #
    #   Will save the DB if both the given number of seconds and the given
    #   number of write operations against the DB occurred.
    #
    #   满足以下条件将会同步数据:
    #   900秒（15分钟）内有1个更改
    #   300秒（5分钟）内有10个更改
    #   60秒内有10000个更改
    #   Note: 可以把所有“save”行注释掉，这样就取消同步操作了
    
    save 900 1
    save 300 10
    save 60 10000
    
    # 指定存储至本地数据库时是否压缩数据，默认为yes，Redis采用LZF压缩，如果为了节省CPU时间，可以关闭该选项，但会导致数据库文件变的巨大
    rdbcompression yes
    
    # 指定本地数据库文件名，默认值为dump.rdb
    dbfilename dump.rdb         
    
*注意*:   
切记注释掉：#daemonize yes 否则无法启动容器
重要话说三遍：注释掉#daemonize yes，注释掉#daemonize yes，注释掉#daemonize yes

## 安装mysql-server

仓库：https://hub.docker.com/r/library/mysql/

#### 拉取镜像：

    docker pull mysql:8.0.13  # 8.0.13为标签tag
    
#### 使用mysql镜像

- 自定义配置文件


    docker run -p 3306:3306 --name mymysql -v $PWD/conf:/etc/mysql/conf.d -v $PWD/logs:/logs -v $PWD/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:8.0.13    

    ------------------------------------
    docker run --name mysql-3910 -p 3910:3910 --restart always --privileged=true -v /server/data/mysql-3910/conf:/etc/mysql/conf.d -v /server/data/mysql-3910/logs:/logs -v /server/data/mysql-3910/data:/var/lib/mysql -e 
    MYSQL_ROOT_PASSWORD=123456 --default-authentication-plugin=mysql_native_password -d mysql:8.0.13 


my.cnf内容

    [mysqld] 
    port = 3910
    character-set-client-handshake = FALSE 
    character-set-server = utf8mb4 
    collation-server = utf8mb4_unicode_ci
    init_connect='SET NAMES utf8mb4'
    default_authentication_plugin = mysql_native_password
    
    [client]
    default-character-set=utf8mb4
    
    [mysql]
    default-character-set=utf8mb4

    
命令说明：

- -p 3306:3306：将容器的 3306 端口映射到主机的 3306 端口。
- -v -v $PWD/conf:/etc/mysql/conf.d：将主机当前目录下的 conf/my.cnf 挂载到容器的 /etc/mysql/my.cnf。看下面补充内容说明。
- -v $PWD/logs:/logs：将主机当前目录下的 logs 目录挂载到容器的 /logs。
- -v $PWD/data:/var/lib/mysql ：将主机当前目录下的data目录挂载到容器的 /var/lib/mysql 。
- -e MYSQL_ROOT_PASSWORD=123456：初始化 root 用户的密码。
- --default-authentication-plugin=mysql_native_password  加上这个客户端才能登录上。在配置文件加了，命令中可已不加。
-  --privileged=true 提升root在docker中的权限，否则只是普通用户
- –restart always：开机启动

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
- 不使用conf配置文件

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
    -d mysql:5.7 \
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
    -d zabbix/zabbix-web-nginx-mysql:centos-4.0.2
    
    ------------------------------ 查看启动日志 ---------------
    docker logs zabbix-web-nginx-mysql

```text
浏览器访问ip:8083查看
默认登录
username:Admin
password:zabbix
```
  
 _注意_ ：生产环境要做数据卷映射。以防止数据丢失。必须加上属性` -e MYSQL_ROOT_PASSWORD="123456"`,否则mysql的`zabbix`用户没有操作mysql数据库的权限。
 
_界面无法选择中文显示的问题_：

打开系统，在用户资料中，可以选择语言，但是无法选择中文。这是由于docker容器没有安装中文字符集，不支持中文。   
因此，要进入容器，安装中文字符集合： 

查看系统当前字符集： `# locale`；     
查看系统可用字符集： `#locale -a`;     
安装中文字符集：  
`yum install kde-l10n-Chinese -y `
`yum -y -q reinstall glibc-common`
 

修改`vim /etc/locale.conf`, LANG=zh_CN.UTF-8;

重新启动容器。再次进入容器，查看系统当前字符集，发现上面修改无效。坑爹……

那是因为centos默认镜像的 /etc/yum.conf 里面有一句`override_install_langs=en_US.utf8`，    
删除这句话，然后重新运行：`yum -y -q reinstall glibc-common`，再重启容器，查看系统当前字符集，可以看到是中文的了。
 
 
       
#### 启动zabbix-agent



    docker run --name zabbix-agent --link zabbix-server-mysql:zabbix-server -d zabbix/zabbix-agent:centos-4.0.2 
    
最后需要在web端将，zabbix-agent添加到zabbix-server的host列表里面。    



####  启动Zabbix Java gateway实例,并关联到zabbix-server  

用于监控jvm/tomcat性能。   
Zabbix监控Java应用程序的关键点在于：配置Zabbix-JavaGateway、让Zabbix-Server能够连接Zabbix-JavaGateway、Tomcat开启JVM远程监控功能等。

    docker run --name zabbix-java-gateway --link zabbix-server-mysql:zabbix-server -d zabbix/zabbix-java-gateway:latest
    

## 安装Jenkins

####　配置日志：   

    [root@ymu /]# mkdir -p /server/data/jenkins
    [root@ymu /]# cd /server/data/jenkins
    [root@ymu jenkins]# touch log.properties
    [root@ymu jenkins]# cat > log.properties <<EOF
    > handlers=java.util.logging.ConsoleHandler
    > jenkins.level=FINEST
    > java.util.logging.ConsoleHandler.level=FINEST
    > EOF


#### 启动jenkins服务

参考：https://github.com/jenkinsci/docker/blob/master/README.md

首先，要对文件夹赋予权限：   
查看文件夹权限：`

    [root@xr-server jenkins]# ls -l /server/data/jenkins/
    total 4
    -rw-r--r--. 1 root root 109 Dec 17 00:55 log.properties
    
赋予文件夹权限：    

    chown -R 1000:1000 /server/data/jenkins/    

启动容器：

    docker run --name ymu-jenkins -p 3001:8080 -p 50000:50000 --env JAVA_OPTS="-Djava.util.logging.config.file=/var/jenkins_home/log.properties" -v /server/data/jenkins:/var/jenkins_home -d jenkins/jenkins:lts

lts：长期支持版本    
以上启动的容器安装Locale插件后，也只能部分中文，原因：未知……
    
说明： 
- -v 会把容器目录`/var/jenkins_home`映射到主机`/server/data/jenkins` 


#### 测试是否安装成功

打开浏览器输入：local:3001

如果看到要求获取登录密码的界面，则成功。恭喜……

获取登录初始密码：   

    [root@ymu secrets]# pwd
    /server/data/jenkins/secrets
    [root@ymu secrets]# cat initialAdminPassword 
    8ccac67d98dd4c77a2c09870e8246e5d
    [root@ymu secrets]#

其它的如和maven的集成等，参考以前文档。  

#### 使用Nginx做方向代理

坑，坑，坑……

一开始按一般的nginx方向代理配置，负载均衡配置，都不行，页面打开异常的慢……

各种百度，不行。还好，想起了google大神……结果找到了：  

https://wiki.jenkins.io/display/JENKINS/Running+Jenkins+behind+Nginx

还是官方文档有用，以后遇到这种问题，都先到官方wiki上找答案才对。  
重新配置：

    upstream jenkins {
      keepalive 32; # keepalive connections
      server 127.0.0.1:9000; # jenkins ip and port
    }
     
    server {
      listen          80;       # Listen on port 80 for IPv4 requests
    
      server_name     ci.xcsqjr.com;
    
      #this is the jenkins web root directory (mentioned in the /etc/default/jenkins file)
      root            /server/data/jenkins/war;
    
      access_log      /server/java/nginx/logs/ci.xcsqjr.com.access.log;
      error_log       /server/java/nginx/logs/ci.xcsqjr.com.error.log;
      ignore_invalid_headers off; #pass through headers from Jenkins which are considered invalid by Nginx server.
    
      location ~ "^/static/[0-9a-fA-F]{8}\/(.*)$" {
        #rewrite all static files into requests to the root
        #E.g /static/12345678/css/something.css will become /css/something.css
        rewrite "^/static/[0-9a-fA-F]{8}\/(.*)" /$1 last;
      }
    
      location /userContent {
        #have nginx handle all the static requests to the userContent folder files
        #note : This is the $JENKINS_HOME dir
        root /server/data/jenkins/war;
        if (!-f $request_filename){
          #this file does not exist, might be a directory or a /**view** url
          rewrite (.*) /$1 last;
    	  break;
        }
    	sendfile on;
      }
    
      location / {
          sendfile off;
          proxy_pass         http://jenkins;
          proxy_redirect     default;
          proxy_http_version 1.1;
    
          proxy_set_header   Host              $host;
          proxy_set_header   X-Real-IP         $remote_addr;
          proxy_set_header   X-Forwarded-For   $proxy_add_x_forwarded_for;
          proxy_set_header   X-Forwarded-Proto $scheme;
          proxy_max_temp_file_size 0;
    
          #this is the maximum upload size
          client_max_body_size       10m;
          client_body_buffer_size    128k;
    
          proxy_connect_timeout      90;
          proxy_send_timeout         90;
          proxy_read_timeout         90;
          proxy_buffering            off;
          #proxy_request_buffering    off; # Required for HTTP CLI commands in Jenkins > 2.54
          proxy_set_header Connection ""; # Clear for keepalive
      }
    
    }

注意：按上面docker安装jenkins的时候，映射了数据卷，所以可以知道：`server/data/jenkins/war` 为jenkins的web根目录。注意配置好。

## 安装gitlab

参考网址：https://docs.gitlab.com/omnibus/docker/



