---
title: MongoDB入门
date: 2021-08-24 09:38:13
categories: mongodb
tags:
---

MongoDB（来自于英文单词“Humongous”，中文含义为“庞大”）是可以应用于各种规模的企业、各个行业以及各类应用程序的开源数据库。基于分布式文件存储的数据库。由C++语言编写。旨在为WEB应用提供可扩展的高性能数据存储解决方案。MongoDB是一个高性能，开源，无模式的文档型数据库，是当前NoSql数据库中比较热门的一种。

MongoDB是一个介于关系数据库和非关系数据库之间的产品，是非关系数据库当中功能最丰富，最像关系数据库的。他支持的数据结构非常松散，是类似json的bjson格式，因此可以存储比较复杂的数据类型。Mongo最大的特点是他支持的查询语言非常强大，其语法有点类似于面向对象的查询语言，几乎可以实现类似关系数据库单表查询的绝大部分功能，而且还支持对数据建立索引。

传统的关系数据库一般由数据库（database）、表（table）、记录（record）三个层次概念组成，MongoDB是由数据库（database）、集合（collection）、文档对象（document）三个层次组成。MongoDB对于关系型数据库里的表，但是集合中没有列、行和关系概念，这体现了模式自由的特点。

MongoDB中的一条记录就是一个文档，是一个数据结构，由字段和值对组成。MongoDB文档与JSON对象类似。字段的值有可能包括其它文档、数组以及文档数组。MongoDB支持OS X、Linux及Windows等操作系统，并提供了Python，PHP，Ruby，Java及C++语言的驱动程序，社区中也提供了对Erlang及.NET等平台的驱动程序。

MySQL的适合对大量或者无固定格式的数据进行存储，比如：日志、缓存等。对事物支持较弱，不适用复杂的多文档（多表）的级联查询。

## mongodb安装

介绍两种安装方式，本机安装方式和docker安装方式。

#### 本机安装

_Centos系统安装：_

1.[下载安装包](https://www.mongodb.com/try/download/community)

{% asset_img a_1.png %}

下载好安装包后解压：
```shell
tar -zxvf mongodb-linux-x86_64-rhel70-5.0.2.tgz 
mv  mongodb-linux-x86_64-rhel70-5.0.2 mongodb-5.0.2
```

2.编辑添加环境变量：
`vim /etc/profile`

```text
####### mongodb #########
MONGODB_HOME=/server/mongodb-5.0.2
export PATH=$MONGODB_HOME/bin:$PATH
```

3.创建数据库目录

默认情况下 MongoDB 启动后会初始化以下两个目录：

- 数据存储目录：/var/lib/mongodb
- 日志文件目录：/var/log/mongodb

我们在启动前可以先创建这两个目录并设置当前用户有读写权限：

```shell
sudo mkdir -p /var/lib/mongo
sudo mkdir -p /var/log/mongodb
sudo chown `whoami` /var/lib/mongo     # 设置权限
sudo chown `whoami` /var/log/mongodb   # 设置权限
```

以上两个路径可以自定义其它地方。但是推荐默认的。

下面启动Mongodb服务:
```shell
mongod --dbpath /var/lib/mongo --logpath /var/log/mongodb/mongod.log --fork
```

查看是否成功启动：
```shell
[root@xr-server-dev mongodb]# tail -10f /var/log/mongodb/mongod.log 
{"t":{"$date":"2021-08-24T02:47:29.352+00:00"},"s":"I",  "c":"FTDC",     "id":20625,   "ctx":"initandlisten","msg":"Initializing full-time diagnostic data capture","attr":{"dataDirectory":"/var/lib/mongo/diagnostic.data"}}
{"t":{"$date":"2021-08-24T02:47:29.353+00:00"},"s":"I",  "c":"STORAGE",  "id":20320,   "ctx":"initandlisten","msg":"createCollection","attr":{"namespace":"local.startup_log","uuidDisposition":"generated","uuid":{"uuid":{"$uuid":"e9217ecb-c1ac-41b8-a0d0-d64a4832f822"}},"options":{"capped":true,"size":10485760}}}
{"t":{"$date":"2021-08-24T02:47:29.375+00:00"},"s":"I",  "c":"INDEX",    "id":20345,   "ctx":"initandlisten","msg":"Index build: done building","attr":{"buildUUID":null,"namespace":"local.startup_log","index":"_id_","commitTimestamp":null}}
{"t":{"$date":"2021-08-24T02:47:29.378+00:00"},"s":"I",  "c":"NETWORK",  "id":23015,   "ctx":"listener","msg":"Listening on","attr":{"address":"/tmp/mongodb-27017.sock"}}
{"t":{"$date":"2021-08-24T02:47:29.378+00:00"},"s":"I",  "c":"NETWORK",  "id":23015,   "ctx":"listener","msg":"Listening on","attr":{"address":"127.0.0.1"}}
{"t":{"$date":"2021-08-24T02:47:29.378+00:00"},"s":"I",  "c":"NETWORK",  "id":23016,   "ctx":"listener","msg":"Waiting for connections","attr":{"port":27017,"ssl":"off"}}
{"t":{"$date":"2021-08-24T02:47:29.428+00:00"},"s":"I",  "c":"CONTROL",  "id":20712,   "ctx":"LogicalSessionCacheReap","msg":"Sessions collection is not set up; waiting until next sessions reap interval","attr":{"error":"NamespaceNotFound: config.system.sessions does not exist"}}
```
看到上面信息说明启动成功了。

_Win系统安装：_

#### docker安装-推荐

方式一：    
推荐该方式
```shell
docker run --restart=always --name mongodb -p 27017:27017  -v /server/dockers/mongodb/db:/data/db  -d mongo --auth
```
`--auth`: 代表要认证登录，否则就是裸奔了。

方式二：指定配置文件

配置文件：{% asset_link mongod.conf %}

报错，为解决……

_设置登录用户:_

1.直接进入`admin`
```shell
[root@xr-server-dev ~]# docker exec -it mongodb mongo admin
MongoDB shell version v5.0.2
connecting to: mongodb://127.0.0.1:27017/admin?compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("2313416c-8534-421b-a2bf-399c7152d41a") }
MongoDB server version: 5.0.2
================
Warning: the "mongo" shell has been superseded by "mongosh",
which delivers improved usability and compatibility.The "mongo" shell has been deprecated and will be removed in
an upcoming release.
We recommend you begin using "mongosh".
For installation instructions, see
https://docs.mongodb.com/mongodb-shell/install/
================
Welcome to the MongoDB shell.
For interactive help, type "help".
For more comprehensive documentation, see
	https://docs.mongodb.com/
Questions? Try the MongoDB Developer Community Forums
	https://community.mongodb.com
> 

```

2.创建用户,此用户创建成功,则后续操作都需要用户认证

```shell
> db.createUser({user:"root",pwd:"root",roles:[{role:'root',db:'admin'}]})
Successfully added user: {
	"user" : "root",
	"roles" : [
		{
			"role" : "root",
			"db" : "admin"
		}
	]
}
> exit; #退出
[root@xr-server-dev ~]#
```

3.试下看能成功连接

直接利用`navicat`客户端连接：

{% asset_img a_2.png %}

## MongoDB 后台管理 Shell

MongoDB 自带的交互式 Javascript shell，用来对 MongoDB 进行操作和管理的交互式环境。 当你进入 mongoDB 后台后，它默认会链接到 test 文档（数据库）：

```shell
[root@xr-server-dev ~]# mongo
MongoDB shell version v5.0.2
connecting to: mongodb://127.0.0.1:27017/?compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("07c8c289-7bc9-48a4-ad1c-1b4ec0a1c711") }
MongoDB server version: 5.0.2
================
Warning: the "mongo" shell has been superseded by "mongosh",
which delivers improved usability and compatibility.The "mongo" shell has been deprecated and will be removed in
an upcoming release.
We recommend you begin using "mongosh".
For installation instructions, see
https://docs.mongodb.com/mongodb-shell/install/
================
---
The server generated these startup warnings when booting: 
        2021-08-24T02:47:28.267+00:00: Using the XFS filesystem is strongly recommended with the WiredTiger storage engine. See http://dochub.mongodb.org/core/prodnotes-filesystem
        2021-08-24T02:47:29.339+00:00: Access control is not enabled for the database. Read and write access to data and configuration is unrestricted
        2021-08-24T02:47:29.339+00:00: You are running this process as the root user, which is not recommended
        2021-08-24T02:47:29.339+00:00: This server is bound to localhost. Remote systems will be unable to connect to this server. Start the server with --bind_ip <address> to specify which IP addresses it should serve responses from, or with --bind_ip_all to bind to all interfaces. If this behavior is desired, start the server with --bind_ip 127.0.0.1 to disable this warning
        2021-08-24T02:47:29.340+00:00: /sys/kernel/mm/transparent_hugepage/enabled is 'always'. We suggest setting it to 'never'
        2021-08-24T02:47:29.340+00:00: /sys/kernel/mm/transparent_hugepage/defrag is 'always'. We suggest setting it to 'never'
        2021-08-24T02:47:29.340+00:00: Soft rlimits for open file descriptors too low
        2021-08-24T02:47:29.340+00:00:         currentValue: 1024
        2021-08-24T02:47:29.340+00:00:         recommendedMinimum: 64000
---
---
        Enable MongoDB's free cloud-based monitoring service, which will then receive and display
        metrics about your deployment (disk utilization, CPU, operation statistics, etc).

        The monitoring data will be available on a MongoDB website with a unique URL accessible to you
        and anyone you share the URL with. MongoDB may use this information to make product
        improvements and to suggest MongoDB products and deployment options to you.

        To enable free monitoring, run the following command: db.enableFreeMonitoring()
        To permanently disable this reminder, run the following command: db.disableFreeMonitoring()
---
>
```

由于是一个JavaScript shell，所以你可以做一些简单的运算：
```shell
> 4 + 4
8
> 2+6
8
> 

```

在命令窗口中停止Mongodb服务：
```shell
> use admin
switched to db admin
> db.shutdownServer()
```

当然，你也可以在非mongodb交互窗口中停止服务：
`mongod --dbpath /var/lib/mongo --logpath /var/log/mongodb/mongod.log --shutdown`

## 学习参考：

[MongoDB教程](https://www.runoob.com/mongodb/mongodb-linux-install.html)

