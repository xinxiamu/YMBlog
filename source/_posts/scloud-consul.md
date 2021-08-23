---
title: Spring Cloud使用Consul做服务注册发现中心
date: 2021-08-23 14:27:40
categories: spring-cloud
tags:
---

Consul 是 HashiCorp 公司推出的开源工具，用于实现分布式系统的服务发现与配置。与其它分布式服务注册与发现的方案，Consul 的方案更“一站式”，内置了服务注册与发现框架、分布一致性协议实现、健康检查、Key/Value 存储、多数据中心方案，不再需要依赖其它工具（比如 ZooKeeper 等）。使用起来也较为简单。Consul 使用 Go 语言编写，因此具有天然可移植性(支持Linux、windows和Mac OS X)；安装包仅包含一个可执行文件，方便部署，与 Docker 等轻量级容器可无缝配合。

## 安装consul服务

[官网](https://learn.hashicorp.com/tutorials/consul/get-started-install?in=consul/getting-started)

方式一：    

参考官网安装步骤即可。

方式二：docker安装-推荐

[参考网址](https://hub.docker.com/_/consul)

```shell
docker search consul
docker pull consul
docker run -d --name=dev-consul -p 8500:8500 -e CONSUL_BIND_INTERFACE=eth0 consul agent -dev -client=0.0.0.0
```
上面是开发模式启动。用于开发环境。如果不是本机启动，别的服务器启动，必须加上配置`-client=0.0.0.0`,否则无法成功注册服务。  

_参数说明：_   
`-client 0.0.0.0`：表明绑定的不是默认的127.0.0.1地址，可以通过公网进行访问。

查看`consul`版本：
```shell
[root@xr-server-dev ~]# docker exec -it consul-dev consul version
Consul v1.10.1
Revision db839f18b
Protocol 2 spoken by default, understands 2 to 3 (agent will automatically use protocol >2 when speaking to compatible agents)
```

查看容器ip：
```shell
[root@xr-server-dev ~]# docker exec -it consul-dev ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:AC:11:00:0F  
          inet addr:172.17.0.15  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:58 errors:0 dropped:0 overruns:0 frame:0
          TX packets:29 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:5925 (5.7 KiB)  TX bytes:24586 (24.0 KiB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:691 errors:0 dropped:0 overruns:0 frame:0
          TX packets:691 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:49379 (48.2 KiB)  TX bytes:49379 (48.2 KiB)

```
`consul-dev`为容器名称

或者：
```shell
docker inspect consul-dev
```

## 生产环境安装Consul

#### 单机模式

[参考网址](https://www.cnblogs.com/brady-wang/p/14440649.html)

_方式一：_使用`docker-compose`搭建集群

>1.集群包含三个server：node1, node2, node3
2.集群包含一个client：node4；并且在client上提供web UI访问服务

1.编辑docker-compose.yaml文件
```shell
version: '2'
networks:
  byfn:

services:
  consul1:
    image: consul
    container_name: node1
    command: agent -server -bootstrap-expect=3 -node=node1 -bind=0.0.0.0 -client=0.0.0.0 -datacenter=dc1
    networks:
      - byfn

  consul2:
    image: consul
    container_name: node2
    command: agent -server -retry-join=node1 -node=node2 -bind=0.0.0.0 -client=0.0.0.0 -datacenter=dc1
    depends_on:
        - consul1
    networks:
      - byfn

  consul3:
    image: consul
    container_name: node3
    command: agent -server -retry-join=node1 -node=node3 -bind=0.0.0.0 -client=0.0.0.0 -datacenter=dc1
    depends_on:
        - consul1
    networks:
      - byfn

  consul4:
    image: consul
    container_name: node4
    command: agent -retry-join=node1 -node=ndoe4 -bind=0.0.0.0 -client=0.0.0.0 -datacenter=dc1 -ui 
    ports:
      - 8500:8500
    depends_on:
        - consul2
        - consul3
    networks:
      - byfn
```

2.启动：
```shell
$ docker-compose up
$ docker exec -t node1 consul members
Node   Address          Status  Type    Build  Protocol  DC   Segment
node1  172.21.0.2:8301  alive   server  1.4.0  2         dc1  <all>
node2  172.21.0.4:8301  alive   server  1.4.0  2         dc1  <all>
node3  172.21.0.3:8301  alive   server  1.4.0  2         dc1  <all>
ndoe4  172.21.0.5:8301  alive   client  1.4.0  2         dc1  <default>
```

#### 集群模式

[参考网址](https://www.cnblogs.com/user-sunli/p/14630784.html)


## Spring Cloud集成Consul

[下载demo](https://github.com/xinxiamu/mu-spring-cloud-sample/tree/master/spring-cloud-consul)

1.服务和消费端加上依赖：
```shell
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-consul-discovery</artifactId>
</dependency>
```

2.在启动类中加上注解：    
`@EnableDiscoveryClient`

3.配置.yml

消费者的：
```shell
server:
  port: 8001
spring:
  application:
    name: consul-service
  cloud:
    consul:
      port: 8500
      host: 192.168.0.3
      discovery:
        service-name: ${spring.application.name}
        prefer-ip-address: true
#        instance-id: ${spring.application.name}-id
```

消费者的：
```shell
server:
  port: 8080
spring:
  application:
    name: consul-consumer
  cloud:
    consul:
      host: 192.168.0.3
      port: 8500
      discovery:
        service-name: ${spring.application.name}
        register: false #不注册到consul
```
