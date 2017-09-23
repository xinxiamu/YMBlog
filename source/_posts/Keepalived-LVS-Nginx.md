---
title: Keepalived+LVS+Nginx负载均衡之高可用
date: 2017-09-23 09:48:19
categories: Nginx
tags: nginx负载均衡
---
### LVS项目介绍
[参考](http://www.linuxvirtualserver.org/zh/lvs1.html) [官网](http://www.linuxvirtualserver.org/)

#### 为什么要使用LVS+Nginx
在用nginx+tomcat做负载均衡时，接收到客户端请求后，nginx会将请求负载转发到tomcat服务端，同时保持和客户端连接，当服务端处理完毕后nginx再将结果返回给客户端。那么就是说，客户端所有的流量都是要经过nginx的，这就造成了一个问题，系统架构中nginx出现了单机性能瓶颈。
而LVS做负载的时候，LVS接到客户端请求,将请求负载转发出去，同时*断开与客户端连接*，服务端处理完毕直接将结果返回给客户端，不再经过LVS。
所以，结合两者优缺点，在nginx前在加多一层LVS为nginx做负载均衡，避免nginx单机性能瓶颈，使系统高可用。
同时，使用Keepalived对LVC做双热备，避免单点故障。

#### LVS搭建
