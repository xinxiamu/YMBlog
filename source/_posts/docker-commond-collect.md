---
title: docker常用命令收藏
date: 2019-01-03 14:31:42
categories: docker
tags:
---

## 容器

### 查看容器端口映射

    [root@sqjr-client-demo-server1-hn zookeeper]# docker port zk-server1
    2181/tcp -> 0.0.0.0:2181
    2888/tcp -> 0.0.0.0:2888
    3888/tcp -> 0.0.0.0:3888
    
### 强制删除运行中的容器

    docker rm -f 容器名称    