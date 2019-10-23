---
title: 利用nexus搭建docker私服
date: 2018-12-26 19:31:05
categories: docker
tags:
---

本文介绍利用Nexus来做docker私服，来管理自己或者公司的镜像。类似于做maven私服一样……

首先要安装nexus：参考章节{%post_link docker-app-install%}，docker安装nexus。  

## 创建Docker仓库

- 首先登录nexus3 。
- 点击“Create repository”按钮，创建仓库。Nexus支持多种仓库类型，例如：maven、npm、docker等。本文创建一个docker仓库。一般来说，对于特定的仓库类型（例如docker），细分了三类，分别是proxy、hosted、group，含义如下：     
  hosted，本地代理仓库，通常我们会部署自己的构件到这一类型的仓库，可以push和pull。   
  proxy，代理的远程仓库，它们被用来代理远程的公共仓库，如maven中央仓库，只能pull。   
  group，仓库组，用来合并多个hosted/proxy仓库，通常我们配置maven依赖仓库组，只能pull。   
  
- 下面创建仓库：

{%asset_img a-1.png%}

填写配置(注意端口号 )：
  
{%asset_img a-2.png%}


## 连接仓库

在其它机子或者本机，连接了仓库，才能做push、pull动作。

- 首先，编辑`vim /etc/docker/daemon.json`


    [root@api data]# cat /etc/docker/daemon.json 
    {
      "insecure-registries" : [
        "11.148.41.11:8082"
      ]
    }
    
`11.148.41.11:8082`：这里ip是nexus服务器的ip，端口是是上面配置的docker仓库的端口。

然后重启docker引擎。

- 登录


    docker login -u admin -p admin123 ip:8082  #注意这里的端口是配置仓库时选择的端口号
    
_登录报错处理：_

1.错误：

    [root@xr-server-dev eureka-server]# docker login  -u admin -p admin123 localhost:8082
    WARNING! Using --password via the CLI is insecure. Use --password-stdin.
    Error response from daemon: login attempt to http://localhost:8082/v2/ failed with status: 401 Unauthorized

错误处理：

{%asset_img j-1.png%}         
    
## 上传镜像

不能直接上传镜像：`docker push nginx:latest`。因为docker默认是上传到docker hub仓库的。

所以要先改镜像标签：

    docker tag nginx:latest ip:8082/nginx:0.1
    
然后上传：
    
    docker push ip:8082/nginx:0.1
    
下面成功截图：

{%asset_img a-3.png%}   

## 拉取镜像

    docker pull ip:8082/nginx:0.1
    
## 搜索镜像

    docker search ip:8082/nginx      
    
    