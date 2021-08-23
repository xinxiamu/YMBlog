---
title: docker使用错误收集
date: 2019-01-23 15:41:18
categories: docker
tags:
---

##  docker 端口映射错误解决方法

错误描述：

    COMMAND_FAILED: '/sbin/iptables -t nat -A DOCKER -p tcp -d 0/0 --dport 8111 -j DNAT --to-destination 172.17.0.6:8111 ! -i docker0' failed: iptables: No chain/target/match by that name.
   
解决：    
 
依次执行以下命令：  
 
    pkill docker
   
    iptables -t nat -F
    ifconfig docker0 down
    brctl delbr docker0   
    
然后重启docker引擎。`systemctl restart docker.service`    

## Docker拉取镜像失败报错Error response from daemon: Get https://registry-1.docker.io

解决方法：

1.打开 vim /etc/docker/daemon.json（若没有自行创建）

2.打开 vim /etc/docker/daemon.json（若没有自行创建）
```shell
{
    "registry-mirrors":["https://docker.mirrors.ustc.edu.cn"]
}
```

或者添加多个：
```shell
# 也可以添加多个国内源
{
 
"registry-mirrors": ["http://hub-mirror.c.163.com", "https://registry.docker-cn.com"]
 
}
```

可用的镜像地址很多，推荐使用阿里云镜像代理。使用如下：

访问如下网址：

https://cr.console.aliyun.com

登录，然后就能看到。

重启一下docker：
```shell
systemctl daemon-reload
systemctl restart docker
```

