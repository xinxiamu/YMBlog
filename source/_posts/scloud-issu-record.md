---
title: Spring Cloud使用问题记录
date: 2019-05-31 18:20:21
categories: spring-cloud
tags:
---

## Zuul问题

1、max-threads：需要根据具体的硬件环境进行调整
2、max-threads：并不是线程数越大越好，线程数增加会增加内存开销同时大量线程切换会浪费不少时间，并且容易造成内存溢出
3、如果单一网关不能满足并发量，网关可以集群，使用nginx分发请求给多个网关
4、可以使用spring-session、redis解决session一致性问题

5.header信息路由到服务后丢失

解决问题
 
- 保留请求header信息
zuul.sensitive-headers=
或者
zuul.routes.xxx.sensitive-headers=
zuul.routes.xxx.custom-sensitive-headers=true

参考这篇文章

Spring Cloud实战小贴士：Zuul处理Cookie和重定向

http://blog.csdn.net/dream8062/article/details/71169628

6.http://blog.didispace.com/spring-cloud-zuul-cookie-redirect/