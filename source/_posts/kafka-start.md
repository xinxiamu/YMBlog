---
title: kafka入门
date: 2020-10-19 11:39:15
categories: kafka
tags:
---

介绍kafka的安装，入门基础概念……

## 方式一 环境安装（官网,推荐）

http://kafka.apache.org/quickstart

### 第一步：获取安装包

[下载](https://www.apache.org/dyn/closer.cgi?path=/kafka/2.6.0/kafka_2.13-2.6.0.tgz)并解压

```shell script
$ tar -xzf kafka_2.13-2.6.0.tgz
$ cd kafka_2.13-2.6.0
```

### 第二步：启动KAFKA环境

_注意：_要求 java 8+

先启动zookeeper：
```shell script
# Start the ZooKeeper service
# Note: Soon, ZooKeeper will no longer be required by Apache Kafka.
$ bin/zookeeper-server-start.sh config/zookeeper.properties
```

打开新的命令行窗口再启动：
```shell script
# Start the Kafka broker service
$ bin/kafka-server-start.sh config/server.properties
```

后台启动方式：

```shell script
$ bin/zookeeper-server-start.sh -daemon config/zookeeper.properties
```

```shell script
$ bin/kafka-server-start.sh -daemon config/server.properties
```

当以上两个服务正确启动，表明你已经具有基础的kafka的运行环境。

### 第三步：创建主题并存储事件（消息、记录）

非常简单，一个主题就类似电脑文件系统里面的一个文件夹，事件就类似文件夹里面的文件。

在你要写入、读取事件之前，你必须首先创建主题。打开另外一个窗口，执行下面命令创建主题：

```shell script
$ bin/kafka-topics.sh --create --topic quickstart-events --bootstrap-server localhost:9092
```
以上命令创建了主题`quickstart-events`。

查看主题命令：
```shell script
$ bin/kafka-topics.sh --describe --topic quickstart-events --bootstrap-server localhost:9092
```
### 第四步：往主题中写入事件

Kafka客户端通过网络与Kafka broker进行通信，以写入（或读取）事件。 一旦收到，broker将以持久和容错的方式存储事件，只要您需要甚至可以永久存储。

打开新的窗口,执行下面生产者客户端：
```shell script
$ bin/kafka-console-producer.sh --topic quickstart-events --bootstrap-server localhost:9092
This is my first event
This is my second event
```
任何时候你可以`Ctrl-C`停止客户端。

### 第五步：消费读取事件

打开新的窗口，执行下面消费端命令：
```shell script
$ bin/kafka-console-consumer.sh --topic quickstart-events --from-beginning --bootstrap-server localhost:9092
This is my first event
This is my second event
```
任何时候你可以`Ctrl-C`停止客户端。

实验：你可以在生产者端，随意输入些东西看看，回到消费者端，你会马上就能看到。

因为事件被持久地存储在Kafka中，所以您可以根据需要任意多次地读取它们。 您可以通过打开另一个终端会话并再次重新运行上一个命令来轻松地验证这一点。

### 第六步：通过KAFKA CONNECT将数据作为事件流导入/导出

在诸如关系数据库或传统消息传递系统之类的现有系统中，您可能拥有大量数据，以及已经使用这些系统的许多应用程序。 通过Kafka Connect，您可以将来自外部系统的数据连续地吸收到Kafka中，反之亦然。 因此，将现有系统与Kafka集成非常容易。 为了使此过程更加容易，有数百种此类连接器可供使用。

看一下Kafka Connect部分，了解更多有关如何连续地将数据导入和导出Kafka的信息。

### 第七步：使用kafka流处理您的活动

一旦将数据作为事件存储在Kafka中，就可以使用Java / Scala的Kafka Streams客户端库处理数据。 它允许您实现关键任务实时应用程序和微服务，其中输入和/或输出数据存储在Kafka主题中。 Kafka Streams结合了在客户端编写和部署标准Java和Scala应用程序的简便性以及Kafka服务器端集群技术的优势，使这些应用程序具有高度可伸缩性，弹性，容错性和分布式性。 该库支持一次精确处理，有状态操作和聚合，开窗，联接，基于事件时间的处理等等。

### 第八步：终止Kafka运行环境

1.终止生产者端和消费者端，通过`Ctrl-C`。
2.终止kafka broker。
3.终止ZooKeeper。

如果您还想删除本地Kafka环境的任何数据，包括您在此过程中创建的所有事件，请运行以下命令：
```shell script
$ rm -rf /tmp/kafka-logs /tmp/zookeeper
```

=============================== 《完》 ====================================

## 方式二 环境安装（单机）

环境要求：

- 系统： Linux CentOS7
- JAVA: jdk_8 +

### 安装JAVA

这个不会安装，回家种番薯……

### 安装Zookeeper（单机模式）

```shell script
# tar -zxf zookeeper-3.4.6.tar.gz
# mv zookeeper-3.4.6 /usr/local/zookeeper
# mkdir -p /var/lib/zookeeper
# cat > /usr/local/zookeeper/conf/zoo.cfg << EOF
> tickTime=2000
> dataDir=/var/lib/zookeeper
> clientPort=2181
> EOF
# export JAVA_HOME=/usr/java/jdk1.8.0_51
# /usr/local/zookeeper/bin/zkServer.sh start
JMX enabled by default
Using config: /usr/local/zookeeper/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
#
```

_Docker环境下安装_

https://hub.docker.com/_/zookeeper

```shell script
# cat > /server/dockers/zookeeper/conf/zoo.cfg << EOF
> tickTime=2000
> dataDir=/tmp/zookeeper
> clientPort=2181
> EOF
```

```shell script
docker run --name zookeeper -p 2181:2181 --restart always -d -v /server/dockers/zookeeper/conf/zoo.cfg:/conf/zoo.cfg zookeeper
```

### 安装 Kafka Broker

成功启动zookeeper后，再安装kafka broker。


安装下面安装kafka在目录`/usr/local/kafka`, 配置`zookeeper`文件，并把kafka元数据存储在`/tmp/kafka-logs`:

```shell script
# tar -zxf kafka_2.11-0.9.0.1.tgz
# mv kafka_2.11-0.9.0.1 /usr/local/kafka
# mkdir /tmp/kafka-logs
# export JAVA_HOME=/usr/java/jdk1.8.0_51
# /usr/local/kafka/bin/kafka-server-start.sh -daemon
/usr/local/kafka/config/server.properties
#
```
