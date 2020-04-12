---
title: RabbitMQ学习-基础
date: 2020-04-11 16:31:59
categories: rabbitmq
tags:
---

## 什么是RabbitMQ？

RabbitMQ是由Erlang语言编写的实现了高级消息队列协议（AMQP）的开源消息代理软件（也可称为 面向消息的中间件）。 

RabbitMQ就像一个邮局，用来投递消息，接收消息。

## 使用场景

1.异步处理，提高并发量，提高程序响应速度。  
2.多个程序之间解耦。多个系统之间通过消息中间件进行消息交互。

典型的应用场景：

1、注册时发送邮件或发送短信
2、日志分析使用，多个服务产生的数据发送到中间件发送到分析服务。
3、消息复制，用于跨机房数据传输、搜索、离线数据计算等。
4、延迟消息发送和暂存，把中间件当成可靠的消息暂存地。接受消息，暂时先不发送。

## 重要概念

{%asset_img a.png%}

（1）_Broker_：经纪人。提供一种传输服务，维护一条从生产者到消费者的传输线路，保证消息数据能按照指定的方式传输。粗略的可以将图中的RabbitMQ Server当作Broker。

（2）_Exchange_：消息交换机。指定消息按照什么规则路由到哪个队列Queue。

（3）_Queue_：消息队列。消息的载体，每条消息都会被投送到一个或多个队列中。

（4）_Binding_：绑定。作用就是将Exchange和Queue按照某种路由规则绑定起来。

（5）_RoutingKey_：路由关键字。Exchange根据RoutingKey进行消息投递。

（6）_Vhost_：虚拟主机。一个Broker可以有多个虚拟主机，用作不同用户的权限分离。一个虚拟主机持有一组Exchange、Queue和Binding。

（7）_Producer_：消息生产者。主要将消息投递到对应的Exchange上面。一般是独立的程序。

（8）_Consumer_：消息消费者。消息的接收者，一般是独立的程序。

（9）_Channel_：消息通道，也称信道。在客户端的每个连接里可以建立多个Channel，每个Channel代表一个会话任务。

## RabbitMQ使用过程概述

AMQP模型中，消息在Producer中产生，并发送到MQ的exchange上，exchange根据配置的路由将消息投递到对应的Queue上，Queue又将消息发送给已经在此Queue上注册的consumer，消息从queue到consumer有push和pull两种方式。

消息队列的使用过程大概如下：

（1）客户端连接到消息队列服务器，打开一个channel。

（2）客户端声明一个exchange，并设置相关属性。

（3）客户端声明一个queue，并设置相关属性。

（4）客户端使用routing key，在exchange和queue之间建立好Binding关系。

（5）生产者客户端投递消息到exchange。

（6）exchange接收到消息后，就根据消息的RoutingKey和已经设置的binding，进行消息路由（投递），将消息投递到一个或多个队列里。

（7）消费者客户端从对应的队列中获取并处理消息。


#### 工作过程

- _生产者客户端_：  
 
1.客户端连接到RabbitMQ服务器上，打开一个消息通道（channel）。         
2.客户端声明一个消息交换机（exchange），并设置相关属性。              
3.客户端声明一个消息队列（queue），并设置相关属性。       
4.客户端使用routing key在消息交换机（exchange）和消息队列（queue）中建立好绑定关系。        
5.客户端投递消息都消息交换机（exchange）上。          
6.客户端关闭消息通道（channel）以及和服务器的连接。      

- _服务器端_：

exchange接收到消息后，根据消息的key（这个key的产生规则暂时没研究，有知道的小伙伴可以留言告诉我）和以及设置的binding，进行消息路由，将消息投递到一个或多个消息队列中。

关于exchange也有几个类型：

(1). Direct交换机：完全根据key进行投递。例如，绑定时设置了routing key为abc，客户端提交信息提交信息时只有设置了key为abc的才会投递到队列；

(2).Topic交换机：在key进行模式匹配后进行投递。例如：符号”#”匹配一个或多个字符，符号”*”匹配一串连续的字母字符，例如”abc.#”可以匹配”abc.def.ghi”，而”abc.*”只可以匹配”abc.def”。

(3).Fanout交换机：它采取广播模式，消息进来时，将会被投递到与改交换机绑定的所有队列中。

- _消费者_

1、消费者和Broker建立TCP连接

2、消费者和Broker建立通道

3、消费者监听指定的Queue（队列）

4、当有消息到达Queue时Broker默认将消息推送给消费者。

5、消费者接收到消息。

6、ack回复

## RabbitMQ的消息持久化

RabbitMQ支持数据持久化，也就是把数据写在磁盘上，可以增加数据的安全性。消息队列持久化包括三个部分：

消息交换机（exchange）持久化，在声明时指定durable为1
消息队列（queue）持久化，在声明时指定durable为1
消息持久化，在投递时指定delivery_mode为2（1是非持久化）
如果消息交换机（exchange）和消息队列（queue）都是持久化的话，那么他们之间的绑定（Binding）也是持久化的。如果消息交换机和消息队列之间一个持久化、一个非持久化，那么就不允许绑定。




