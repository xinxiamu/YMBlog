---
title: centos6安装docker-ce
date: 2018-12-18 16:00:19
categories: docker
tags:
---

Docker 运行在 CentOS 7 上，要求系统为64位、系统内核版本为 3.10 以上。
Docker 运行在 CentOS-6.5 或更高的版本的 CentOS 上，要求系统为64位、系统内核版本为 2.6.32-431 或者更高版本

CentOS 7 的内核一般都是3.10的，而CentOS 6.X 的内核一般都是2.6，在2.6的内核下，Docker运行会比较卡，所以一般会选择升级到3.10版本。

centos升级内核方法：{% post_link  centos6-kernel-update Linux升级内核 %}


1.安装docker

    [root@localhost ~]# yum install docker-io

2.查看Docker版本信息

    [root@iZwz9b0bqrkbhqd5lu2pwhZ ~]# docker version
    Client version: 1.7.1
    Client API version: 1.19
    Go version (client): go1.4.2
    Git commit (client): 786b29d/1.7.1
    OS/Arch (client): linux/amd64
    Get http:///var/run/docker.sock/v1.19/version: dial unix /var/run/docker.sock: no such file or directory. Are you trying to connect to a TLS-enabled daemon without TLS?

3.启动Docker

    [root@iZwz9b0bqrkbhqd5lu2pwhZ ~]# service docker start
    Starting cgconfig service:                                 [  OK  ]
    Starting docker:	                                   [  OK  ]


4.查看Docker日志

    [root@iZwz9b0bqrkbhqd5lu2pwhZ ~]# cat /var/log/docker
    \nTue Dec 18 16:19:04 CST 2018\n
    time="2018-12-18T16:19:04.578091169+08:00" level=info msg="Listening for HTTP on unix (/var/run/docker.sock)" 
    time="2018-12-18T16:19:06.456619838+08:00" level=warning msg="Running modprobe bridge nf_nat failed with message: install /sbin/modprobe --ignore-install bridge && /sbin/sysctl -q -w net.bridge.bridge-nf-call-arptables=0 net.bridge.bridge-nf-call-iptables=0 net.bridge.bridge-nf-call-ip6tables=0\ninsmod /lib/modules/4.4.167-1.el6.elrepo.x86_64/kernel/net/netfilter/nf_conntrack.ko \ninsmod /lib/modules/4.4.167-1.el6.elrepo.x86_64/kernel/net/netfilter/nf_nat.ko \n, error: exit status 1" 
    time="2018-12-18T16:19:06.713397637+08:00" level=info msg="Loading containers: start." 
    
    time="2018-12-18T16:19:06.713664810+08:00" level=info msg="Loading containers: done." 
    time="2018-12-18T16:19:06.713684883+08:00" level=info msg="Daemon has completed initialization" 
    time="2018-12-18T16:19:06.713703135+08:00" level=info msg="Docker daemon" commit="786b29d/1.7.1" execdriver=native-0.2 graphdriver=devicemapper version=1.7.1 

5.停止Docker

    [root@localhost ~]# service docker stop
    
6.卸载Docker

6.1  查看安装的Docker包

    [root@localhost ~]# yum list installed | grep docker
    
6.2 yum 卸载 Docker

    [root@localhost ~]# yum -y remove docker-io.x86_64
    
6.3 删除Docker镜像

    [root@localhost ~]# rm -rf /var/lib/docker       
    

