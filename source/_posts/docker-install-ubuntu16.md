---
title: Ubuntu16.0.4安装docker-ce
date: 2018-02-28 09:55:22
categories: docker
tags: docker ce安装
---

本文介绍在Ubuntu环境下安装Docker ce……

参考：https://docs.docker.com/install/linux/docker-ce/ubuntu/

##  卸载旧版本：

    $ sudo apt-get remove docker docker-engine docker.io

## 安装系统可选内核

参考：
https://docs.docker.com/install/linux/docker-ce/ubuntu/#supported-storage-drivers

## 使用 APT 安装

- *更新系统包*


    $ sudo apt-get update
    
    
- *添加ca证书*

由于apt源使用HTTPS以确保软件下载过程中不被篡改。因此,我们首先需要添加使用HTTPS传输的软件包以及CA证书。

    
    $ sudo apt-get install \
        apt-transport-https \
        ca-certificates \
        curl \
        software-properties-common    
   
如果出现错误：

    E: Could not get lock /var/lib/dpkg/lock - open (11: Resource temporarily unavailable)
    E: Unable to lock the administration directory (/var/lib/dpkg/), is another process using it?

执行下面命令：

    mutian@mutian-ThinkPad-T440p:~$ sudo rm /var/cache/apt/archives/lock
    mutian@mutian-ThinkPad-T440p:~$ sudo rm /var/lib/dpkg/lock 
    
然后再重试。


- *添加软件源的GPG密钥*

为了确认所下载软件包的合法性,需要添加软件源的GPG密钥。

    $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    

验证密钥：

    mutian@mutian-ThinkPad-T440p:~$ sudo apt-key fingerprint 0EBFCD88
    pub   4096R/0EBFCD88 2017-02-22
          Key fingerprint = 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
    uid                  Docker Release (CE deb) <docker@docker.com>
    sub   4096R/F273FCD8 2017-02-22
    
    mutian@mutian-ThinkPad-T440p:~$ 


- *添加Docker软件源* 

然后,我们需要向source.list	中添加Docker软件源

    $ sudo add-apt-repository \
       "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
       $(lsb_release -cs) \
       stable"
       
>以上命令会添加稳定版本的	Docker	CE	APT	镜像源,如果需要最新或者测试版本的
Docker	CE	请将	stable	改为	edge	或者	test。从	Docker	17.06	开始,edge	test	版本的
APT	镜像源也会包含稳定版本的	Docker。 

- *安装DOCKER CE*

1.更新系统包

    $ sudo apt-get update
    
2.安装

    $ sudo apt-get install docker-ce    


## 镜像加速
      
国内从Docker	Hub拉取镜像有时会遇到困难,此时可以配置镜像加速器。Docker	官方和国
内很多云服务商都提供了国内加速器服务,例如:   

- [Docker官方提供的中国registry mirror](https://docs.docker.com/registry/recipes/mirror/#use-case-the-china-registry-mirror)
- [阿里云加速器](https://cr.console.aliyun.com/#/accelerator)
- [DaoCloud	加速器](https://www.daocloud.io/mirror#accelerator-doc) 

我们以Docker官方加速器为例进行介绍。

Ubuntu16.04+、Debian	8+、CentOS7环境下：

对于使用	systemd	的系统,请在/etc/docker/daemon.json中写入如下内容(如果文件不存
在请新建该文件)

    {
    		"registry-mirrors":	[
    				"https://registry.docker-cn.com"
    		]
    }
    
>注意,一定要保证该文件符合json规范,否则Docker将不能启动。 

之后重新启动服务。

    $	sudo systemctl daemon-reload
    $	sudo systemctl restart docker
    

## 测试安装是否成功

    $ sudo docker run hello-world
    

如图出现则表示安装成功：

{% asset_img a.png %}    
    
           