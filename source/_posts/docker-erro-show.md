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

