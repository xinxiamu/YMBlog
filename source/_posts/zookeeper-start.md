---
title: zookeeper使用入门
date: 2018-06-28 10:00:46
categories: zookeeper
tags: 
---

## docker安装zookeeper（推荐）

参考:https://hub.docker.com/_/zookeeper

### 1.拉取最新镜像

    docker pull zookeeper
    
### 2.创建配置文件zoo.cfg
 
 首先启动一个zk的一个临时容器，用来从容器中拷贝原始配置文件zoo.cfg。拷贝完后，强制删除容器。
 
    docker run --name temp-zookeeper  -d zookeeper
    docker cp temp-zookeeper:/conf/zoo.cfg /server/data/zookeeper/
    docker rm -f temp-zookeeper
    
### 3.按配置文件启动新的可用zk服务
 
    docker run --name zk-server1 --restart always -p 2181:2181 -p 2888:2888 -p 3888:3888 -d -v /server/data/zookeeper/zoo.cfg:/conf/zoo.cfg zookeeper
    
查看启动日志：

    docker logs zk-server1
    
### 4.测试是否成功
 
- 通zkCli连接zk server
 
 
    docker run -it --rm --link zk-server1:zookeeper zookeeper zkCli.sh -server zookeeper 
    
    -----------------------------
    [root@sqjr-client-demo-server1-hn zookeeper]# docker run -it --rm --link zk-server1:zookeeper zookeeper zkCli.sh -server zookeeper
    Connecting to zookeeper
    2019-01-03 06:37:36,491 [myid:] - INFO  [main:Environment@100] - Client environment:zookeeper.version=3.4.13-2d71af4dbe22557fda74f9a9b4309b15a7487f03, built on 06/29/2018 04:05 GMT
    2019-01-03 06:37:36,494 [myid:] - INFO  [main:Environment@100] - Client environment:host.name=c2f781bcaf31
    2019-01-03 06:37:36,495 [myid:] - INFO  [main:Environment@100] - Client environment:java.version=1.8.0_181
    2019-01-03 06:37:36,497 [myid:] - INFO  [main:Environment@100] - Client environment:java.vendor=Oracle Corporation
    2019-01-03 06:37:36,497 [myid:] - INFO  [main:Environment@100] - Client environment:java.home=/usr/lib/jvm/java-1.8-openjdk/jre
    2019-01-03 06:37:36,498 [myid:] - INFO  [main:Environment@100] - Client environment:java.class.path=/zookeeper-3.4.13/bin/../build/classes:/zookeeper-3.4.13/bin/../build/lib/*.jar:/zookeeper-3.4.13/bin/../lib/slf4j-log4j12-1.7.25.jar:/zookeeper-3.4.13/bin/../lib/slf4j-api-1.7.25.jar:/zookeeper-3.4.13/bin/../lib/netty-3.10.6.Final.jar:/zookeeper-3.4.13/bin/../lib/log4j-1.2.17.jar:/zookeeper-3.4.13/bin/../lib/jline-0.9.94.jar:/zookeeper-3.4.13/bin/../lib/audience-annotations-0.5.0.jar:/zookeeper-3.4.13/bin/../zookeeper-3.4.13.jar:/zookeeper-3.4.13/bin/../src/java/lib/*.jar:/conf:
    2019-01-03 06:37:36,498 [myid:] - INFO  [main:Environment@100] - Client environment:java.library.path=/usr/lib/jvm/java-1.8-openjdk/jre/lib/amd64/server:/usr/lib/jvm/java-1.8-openjdk/jre/lib/amd64:/usr/lib/jvm/java-1.8-openjdk/jre/../lib/amd64:/usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib
    2019-01-03 06:37:36,498 [myid:] - INFO  [main:Environment@100] - Client environment:java.io.tmpdir=/tmp
    2019-01-03 06:37:36,498 [myid:] - INFO  [main:Environment@100] - Client environment:java.compiler=<NA>
    2019-01-03 06:37:36,498 [myid:] - INFO  [main:Environment@100] - Client environment:os.name=Linux
    2019-01-03 06:37:36,498 [myid:] - INFO  [main:Environment@100] - Client environment:os.arch=amd64
    2019-01-03 06:37:36,498 [myid:] - INFO  [main:Environment@100] - Client environment:os.version=3.10.0-693.2.2.el7.x86_64
    2019-01-03 06:37:36,498 [myid:] - INFO  [main:Environment@100] - Client environment:user.name=root
    2019-01-03 06:37:36,498 [myid:] - INFO  [main:Environment@100] - Client environment:user.home=/root
    2019-01-03 06:37:36,499 [myid:] - INFO  [main:Environment@100] - Client environment:user.dir=/zookeeper-3.4.13
    2019-01-03 06:37:36,499 [myid:] - INFO  [main:ZooKeeper@442] - Initiating client connection, connectString=zookeeper sessionTimeout=30000 watcher=org.apache.zookeeper.ZooKeeperMain$MyWatcher@4b85612c
    Welcome to ZooKeeper!
    2019-01-03 06:37:36,523 [myid:] - INFO  [main-SendThread(zookeeper:2181):ClientCnxn$SendThread@1029] - Opening socket connection to server zookeeper/172.17.0.5:2181. Will not attempt to authenticate using SASL (unknown error)
    JLine support is enabled
    2019-01-03 06:37:36,595 [myid:] - INFO  [main-SendThread(zookeeper:2181):ClientCnxn$SendThread@879] - Socket connection established to zookeeper/172.17.0.5:2181, initiating session
    [zk: zookeeper(CONNECTING) 0] 2019-01-03 06:37:36,623 [myid:] - INFO  [main-SendThread(zookeeper:2181):ClientCnxn$SendThread@1303] - Session establishment complete on server zookeeper/172.17.0.5:2181, sessionid = 0x1003451023c0000, negotiated timeout = 30000
    
    WATCHER::
    
    WatchedEvent state:SyncConnected type:None path:null
    
    [zk: zookeeper(CONNECTED) 0]
    
- 创建节点
 
 
    [zk: zookeeper(CONNECTED) 6] create /zk myData
    Created /zk 
    
-   获取节点信息

    
    [zk: zookeeper(CONNECTED) 7] get /zk
    myData
    cZxid = 0x2
    ctime = Thu Jan 03 06:48:26 GMT 2019
    mZxid = 0x2
    mtime = Thu Jan 03 06:48:26 GMT 2019
    pZxid = 0x2
    cversion = 0
    dataVersion = 0
    aclVersion = 0
    ephemeralOwner = 0x0
    dataLength = 6
    numChildren = 0

- 删除节点信息


    [zk: zookeeper(CONNECTED) 8] delete /zk
    [zk: zookeeper(CONNECTED) 9] get /zk
    Node does not exist: /zk  
    
## 直接安装

下载地址： https://zookeeper.apache.org/releases.html

1.下载后解压到某个目录下。  
2.修改配置文件，再conf目录下，复制一份`zoo_sample.cfg`,把名字改为`zoo.cfg`。并修改内容：

```shell script
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
dataDir=/tmp/zookeeper
# the port at which the clients will connect
# 监听客户端连接的端口
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1

## Metrics Providers
#
# https://prometheus.io Metrics Exporter
#metricsProvider.className=org.apache.zookeeper.metrics.prometheus.PrometheusMetricsProvider
#metricsProvider.httpPort=7000
#metricsProvider.exportJvmInfo=true

#内嵌的jetty服务器的启动端口，默认8080，不修改的话，可能引起端口冲突
admin.serverPort=8079


```

## 集群配置             
