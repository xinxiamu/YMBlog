---
title: centos下rabbitmq快速安装
date: 2018-01-02 16:02:01
categories: CentOs
tags: centos-rabbitmq-install
---

一、安装erlang
sudo yum install erlang

检查是否安装好：

    [root@localhost /]# erl
    Erlang R16B03-1 (erts-5.10.4) [source] [64-bit] [smp:4:4] [async-threads:10] [hipe] [kernel-poll:false]

二、安装rabbitmq

（1）下载安装包
wget http://www.rabbitmq.com/releases/rabbitmq-server/v3.6.0/rabbitmq-server-3.6.0-1.noarch.rpm

（2）安装
rpm --import https://www.rabbitmq.com/rabbitmq-signing-key-public.asc
yum install rabbitmq-server-3.6.0-1.noarch.rpm

（3）启用web管理插件
rabbitmq-plugins enable rabbitmq_management

三、启动RabbitMQ
chkconfig rabbitmq-server on  //开机启动设置
service rabbitmq-server start

 四、打开对应端口
        # firewall-cmd --permanent --zone=public --add-port=5672/tcp
        # firewall-cmd --permanent --zone=public --add-port=15672/tcp
        # firewall-cmd --reload

五、打开网页
http://119.23.78.160:15672/