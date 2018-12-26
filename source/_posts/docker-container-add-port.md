---
title: docker给运行中的容器添加映射端口
date: 2018-12-26 16:31:53
categories: docker
tags:
---

容器启动的时候，只映射了一些端口，那如果想给运行中的容器映射端口到主机该怎么弄呢……

- 方法一：

1.获取容器id

    docker ps
    docker inspect `container_name` | grep IPAddress
    
    [root@api data]# docker inspect nexus3 | grep IPAddress
                "SecondaryIPAddresses": null,
                "IPAddress": "172.17.0.4",
                        "IPAddress": "172.17.0.4",
    
2.iptable转发端口  

将容器的80端口映射到dockers宿主机的9998端口

   iptables -t nat -A  DOCKER -p tcp --dport 8001 -j DNAT --to-destination 172.17.0.19:8000
    
3.查看docker端口映射

    [root@api docker]# docker port nexus3 
    8081/tcp -> 0.0.0.0:8085  
  
发现没映射成功……悲剧 

无效，未解决……

    
- 方法二：

1.提交一个运行中的容器为镜像

    docker commit containerid foo/live
    
2.运行镜像并添加端口

    docker run -d -p 8000:80  foo/live /bin/bash
    
- 方法三（比方法二方便，推荐）

1.首先stop容器，在宿主机编辑：`/var/lib/docker/containers/[hash_of_the_container]/hostconfig.json` 

添加映射，容器端口8082映射到宿主8888

     "PortBindings": {
        "8081/tcp": [
            {
                "HostIp": "",
                "HostPort": "8085"
            }
        ],
        "8082/tcp": [
            {
                "HostIp": "",
                "HostPort": "8888"
            }
        ]
     }    
     
2.重启docker引擎

    systemctl restart docker.service
    
3.启动容器

    docker start 容器名
    
4.查看容器端口映射

    docker port 容器名                   