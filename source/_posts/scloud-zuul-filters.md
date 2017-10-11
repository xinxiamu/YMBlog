---
title: scloud-zuul-filters网关过滤
date: 2017-10-11 16:56:22
categories: spring-cloud
tags: zuul-filters
---
在一个微服务系统中，多个服务可能都需要做一些同样的非业务层面的鉴权，校验等，如果分散在各个服务中做，将加大维护难度。因此，放到统一网关中做同样的鉴权处理，简化维护。
为了达到这个目的，因此需要在网关层做拦截，过滤。

