---
title: RabbitMQ学习-交换机
date: 2020-04-12 00:18:23
categories: rabbitmq
tags:
---

本文介绍交换机exchange的使用。

## Exchange交换机的类型

RabbitMQ的Exchange（交换器）分为四类：

direct（默认）
headers
fanout
topic

其中headers交换器允许你匹配AMQP消息的header而非路由键，除此之外headers交换器和direct交换器完全一致，但性能却很差，几乎用不到，所以我们本文也不做讲解。

_注意_：fanout、topic交换器是没有历史数据的，也就是说对于中途创建的队列，获取不到之前的消息

## 1、 Direct Exchange

（1）名称：直接交换器类型

（2）默认的预先定义exchange名字：空字符串或者amq.direct

（3）作用描述：根据Binding指定的Routing Key，将符合Key的消息发送到Binding的Queue。可以构建点对点消息传输模型。

{%asset_img a-1.png%}

如图中RoutingKey分别是error、info、warning，其中error被Binding（绑定）到queue1和queue2上，info和warning被Binding到queue2上。当消息的RoutingKey是error，这条消息将被投递到queue1和queue2中（相当于消息被复制成两个分别投放到两个queue中），然后分别被Consumer1和Consumer2处理。如果消息的RoutingKey是info或者warning，这条消息只会被投递到queue2中，然后被Consumer2处理。如果消息的RoutingKey是其他的字符串，这条消息则会被丢弃。

## 2、 fanout交换器——发布/订阅模式

（1）名称：广播式交换器类型

（2）默认的预先定义exchange名字：amq.fanout

（3）作用描述：将同一个message发送到所有同该Exchange 绑定的queue。不论RoutingKey是什么，这条消息都会被投递到所有与此Exchange绑定的queue中。

{%asset_img a-2.png%}

广播式交换器类型的工作方式：不使用任何参数将queue和Exchange进行Binding，发布者publisher向Exchange发送一条消息（注意：直接交换器类型中的producer变成了publisher，其中隐含了两种交换器的区别），然后这条消息被无条件的投递给所有和这个Exchange绑定的queue中。

如图中，没有RoutingKey的限制，只要消息到达Exchange，都会被投递到queue1和queue2中，然后被对应的Consumer处理。

## 3、 topic交换器——匹配订阅模式

（1）名称：主题交换器类型

（2）默认的预先定义exchange名字：amq.topic

（3）作用描述：根据Binding指定的RoutingKey，Exchange对key进行模式匹配后投递到相应的Queue，模式匹配时符号“#”匹配一个或多个词，符号“*”匹配正好一个词，而且单词与单词之间必须要用“.”符号进行分隔。此模式可以用来支持经典的发布/订阅消息传输模型-使用主题名字空间作为消息寻址模式，将消息传递给那些部分或者全部匹配主题模式的queue。

{%asset_img a-3.png%}

 如图中，假如消息的RoutingKey是American.action.13，这条消息将被投递到Q1和Q2中。假如RoutingKey是American.action.13.test（注意：此处是四个词），这条消息将会被丢弃，因为没有routingkey与之匹配。假如RoutingKey是Chinese.action.13，这条消息将被投递到Q2和Q3中。假如RoutingKey是Chinese.action.13.test，这条消息只会被投递到Q3中，#可以匹配一个或者多个单词，而*只能匹配一个词。


## 参考

https://www.cnblogs.com/vipstone/p/9295625.html
