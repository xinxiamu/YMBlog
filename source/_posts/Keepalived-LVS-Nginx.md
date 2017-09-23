---
title: Keepalived+LVS+Nginx负载均衡之高可用
date: 2017-09-23 09:48:19
categories: Nginx
tags: nginx负载均衡
---
### 为什么要使用LVS+Nginx
在用nginx+tomcat做负载均衡时，接收到客户端请求后，nginx会将请求负载转发到tomcat服务端，同时保持和客户端连接，当服务端处理完毕后nginx再将结果返回给客户端。那么就是说，客户端所有的流量都是要经过nginx的，这就造成了一个问题，系统架构中nginx出现了单机性能瓶颈。
而LVS做负载的时候，LVS接到客户端请求,将请求负载转发出去，同时*断开与客户端连接*，服务端处理完毕直接将结果返回给客户端，不再经过LVS。
所以，结合两者优缺点，在nginx前在加多一层LVS为nginx做负载均衡，避免nginx单机性能瓶颈，使系统高可用。
同时，使用Keepalived对LVC做双热备，避免单点故障。

[参考](http://www.linuxvirtualserver.org/zh/lvs1.html) [官网](http://www.linuxvirtualserver.org/)

### Keepalived介绍
Keepalived是分布式部署系统解决系统高可用的软件，结合LVS（Linux Virtual Server）使用，其功能类似于heartbeat，解决单机宕机的问题。
keepalived是以VRRP协议为实现基础的，VRRP全称Virtual Router Redundancy Protocol，即虚拟路由冗余协议。通过VRRP协议结合LVS，对组群服务器监控情况，若master出现宕机情况，则将VIP漂移到backup机上。实现了分布式系统高可用。可以理解为：keepalived是LVS的管理软件，根据监控情况，将宕机服务器从ipvsadm移除掉。

### Keepalived+LVS+Nginx实现系统高可用
#### 1. 架构图
{% asset_img a.png %} 
 
|  服务器   | IP地址 | 说明 |
| :------: | :------:|:-----:|
| 虚拟IP | 192.168.1.120:80 | - |
| 主机 | 192.168.1.104:80 | - |
| 备机 | 192.168.1.103:80 | - |
| Web站点A | 192.168.1.101:8081 | 不同端口 |
| Web站点B | 192.168.1.101:8082	 | 不同端口 |

#### 2. 安装LVS
##### 2.1  安装ipvsadm，实现系统支持LVS
`yum install ipvsadm`
#### 3. 安装Keepalived
`yum install Keepalived`
将keepalived设置开机启动
`systemctl enable keepalived`
##### 3.1   配置keepalived

