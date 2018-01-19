---
title: rabbitmq快速安装
date: 2018-01-02 16:02:01
categories: rabbitmq
tags: rabbitmq-install
---

本文介绍rabbitmq在各系统平台下的安装……

## 在Centos下的快速安装

一、安装erlang
sudo yum install erlang

检查是否安装好：

    [root@localhost /]# erl
    Erlang R16B03-1 (erts-5.10.4) [source] [64-bit] [smp:4:4] [async-threads:10] [hipe] [kernel-poll:false]

二、安装rabbitmq

（1）下载安装包
wget http://www.rabbitmq.com/releases/rabbitmq-server/v3.6.0/rabbitmq-server-3.6.0-1.noarch.rpm

（2）安装

    > rpm --import https://www.rabbitmq.com/rabbitmq-release-signing-key.asc
    > yum install rabbitmq-server-3.6.0-1.noarch.rpm

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

## 在Ubuntu下的快速安装

安装最新版，参考网址： 
http://www.rabbitmq.com/install-debian.html

注：下面安装的不是最新版本。
一. 安装对应erlang版本：
erlang-nox (>= 1:19.3-1) | esl-erlang (>= 1:19.3-1).
`sudo apt-get install erlang-nox`

二. 安装rabbitMq:

    $ sudo apt-get update
    $ sudo apt-get install rabbitmq-server
    
三. 启用web管理插件：

`sudo rabbitmq-plugins enable rabbitmq_management`  

四. 访问
打开：http://localhost:15672  

五. 登录（本机）：
用户：guest
密码：guest

注：guest用户是系统用户，默认是不允许远程登录的。如果是在服务器端安装，需要添加用户才能远程登录。
