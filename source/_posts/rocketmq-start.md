---
title: rocketmq个环境下安装
date: 2021-11-01 17:04:14
categories: RocketMQ
tags:
---

[官网](http://rocketmq.apache.org/)

[Apache RocketMQ开发者指南](https://github.com/apache/rocketmq/tree/master/docs/cn)


## 安装服务

下面介绍在各种系统环境下各服务安装。

### linux下安装

#### 开发环境单机安装

1.下载源码：[源码包](https://www.apache.org/dyn/closer.cgi?path=rocketmq/4.9.3/rocketmq-all-4.9.3-source-release.zip)

2.解压源码包构建：
```shell
> unzip rocketmq-all-4.9.3-source-release.zip
> cd rocketmq-all-4.9.3/
> mvn -Prelease-all -DskipTests clean install -U
> cd distribution/target/rocketmq-4.9.3/rocketmq-4.9.3
```

或者直接下载已经构建好的包：

3.修改配置，允许远程连接：

进入如`/server/rocketmq/rocketmq-4.9.3/conf`
```shell
[root@server-dev-01 conf]# cat broker.conf 
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
#  Unless required by applicable law or agreed to in writing, software
#  distributed under the License is distributed on an "AS IS" BASIS,
#  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#  See the License for the specific language governing permissions and
#  limitations under the License.

brokerClusterName = DefaultCluster
brokerName = broker-a
brokerId = 0
deleteWhen = 04
fileReservedTime = 48
brokerRole = ASYNC_MASTER
flushDiskType = ASYNC_FLUSH
autoCreateTopicEnable=true

namesrvAddr=192.168.10.131:9876
brokerIP1=192.168.10.131

```

实际添加配置：
```text
autoCreateTopicEnable=true #自动创建主题

namesrvAddr=192.168.10.131:9876 #namesrv服务地址
brokerIP1=192.168.10.131 #远程访问ip地址
brokerIP2=127.0.0.1 #内网访问ip地址
```
 
4.启动

注意：修改启动脚本java堆栈参数。默认8g。否则内存不够，无法启动。

启动NameServer：
```shell
> nohup sh bin/mqnamesrv &
> tail -f ~/logs/rocketmqlogs/namesrv.log
The Name Server boot success...
```

启动Broker(指向特定的conf位置文件)：
```shell
> nohup sh bin/mqbroker -n localhost:9876 -c conf/broker.conf &
> tail -f ~/logs/rocketmqlogs/broker.log 
The broker[%s, 172.30.30.233:10911] boot success...
```

4.停止RocketMQ服务

先停止broker服务，再停止namesre

```shell
> sh bin/mqshutdown broker
The mqbroker(36695) is running...
Send shutdown request to mqbroker(36695) OK

> sh bin/mqshutdown namesrv
The mqnamesrv(36664) is running...
Send shutdown request to mqnamesrv(36664) OK
```

#### Dledger模式搭建


### win系统安装

### docker安装

### 安装控制台rocketmq-dashboard

1.下载

[网址](https://github.com/apache/rocketmq-dashboard)

下载：`git clone https://github.com/apache/rocketmq-dashboard.git`

编译得到`jar`包执行：
```shell
mvn clean package -Dmaven.test.skip=true
java -jar target/rocketmq-dashboard-1.0.1-SNAPSHOT.jar
```

注意先修改配置文件，指定正确的namesrv地址。可以把源码导入本地idea配置好，install得到jar包，在放到服务器执行即可。

[修改配置参考](https://github.com/apache/rocketmq-dashboard/blob/master/docs/1_0_0/UserGuide_CN.md)

