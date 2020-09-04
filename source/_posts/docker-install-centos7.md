---
title: centos7安装docker-ce
date: 2017-11-15 13:55:39
categories: docker
tags: centos-docker-install
---

本文介绍docker在centos7系统上的安装。
参考：https://docs.docker.com/engine/installation/linux/docker-ce/centos/#uninstall-old-versions
参考中文：http://docs.docker-cn.com/engine/installation/linux/docker-ce/centos/#%E4%BD%BF%E7%94%A8%E9%95%9C%E5%83%8F%E4%BB%93%E5%BA%93%E8%BF%9B%E8%A1%8C%E5%AE%89%E8%A3%85

## 1. 卸载旧版本docker

    $ sudo yum remove docker \
                        docker-client \
                        docker-client-latest \
                        docker-common \
                        docker-latest \
                        docker-latest-logrotate \
                        docker-logrotate \
                        docker-engine

---
    [root@iZj6ca50pk1lwxqo14jss8Z ~]# sudo yum remove docker \
    >                   docker-common \
    >                   docker-selinux \
    >                   docker-engine
    Loaded plugins: fastestmirror
    No Match for argument: docker
    No Match for argument: docker-common
    No Match for argument: docker-selinux
    No Match for argument: docker-engine
    No Packages marked for removal
    
    ---
    表明没有旧版本
    
旧版本的会安装在`/var/lib/docker/`，包括images，images, containers, volumes, 和 networks。docker ce现在命名为docker-ce。
 
## 2. 安装docker-ce

共有三种方式安装,根据自己喜欢方式选择一种安装：
- 配置安装源，从安装源拉取安装。推荐，但是网络要好
- 下载安装包，执行安装。网络不好，采用。
- 下载脚本执行安装。开发环境这种方式方便。

### 2.1 repository方式安装，推荐
第一次安装，需要先安装Docker repository,然后就可以从repository安装docker或者更新docker。
1. 安装依赖包


    $ sudo yum install -y yum-utils \
      device-mapper-persistent-data \
      lvm2


2. 安装源
        
        
    $ sudo yum-config-manager \
        --add-repo \
        https://download.docker.com/linux/centos/docker-ce.repo                   
                      
3. 可选: 开启edge和test源


    $ sudo yum-config-manager --enable docker-ce-edge
    $ sudo yum-config-manager --enable docker-ce-test                     
    -----
    关闭：
    $ sudo yum-config-manager --disable docker-ce-edge    
    
4. 安装docker

4.1 安装最新版本

    $ sudo yum install docker-ce docker-ce-cli containerd.io
    
> Warning: If you have multiple Docker repositories enabled, installing or updating without specifying a version in the yum install or yum update command will always install the highest possible version, which may not be appropriate for your stability needs.
> 安装报错： 查看下面收集安装异常信息。

4.2 安装指定版本
在生成环境，有时候要安装指定版本。

- 查看所有可用版本

        $ yum list docker-ce --showduplicates | sort -r
        
        docker-ce.x86_64            17.09.ce-1.el7.centos             docker-ce-stable                     

第二列是版本号。        
The contents of the list depend upon which repositories are enabled, and will be specific to your version of CentOS (indicated by the .el7 suffix on the version, in this example). Choose a specific version to install. The second column is the version string. You can use the entire version string, but you need to include at least to the first hyphen. The third column is the repository name, which indicates which repository the package is from and by extension its stability level. To install a specific version, append the version string to the package name and separate them by a hyphen (-).
> Note: The version string is the package name plus the version up to the first hyphen. In the example above, the fully qualified package name is docker-ce-17.06.1.ce.

    $ sudo yum install docker-ce-<VERSION>
    如： sudo yum install docker-ce- 17.09.ce-1.el7.centos
    
4.3 启动docker                 
    
    $ sudo systemctl start docker                      

4.4 验证是否安装成功

    $ sudo docker run hello-world
    
会下载docker镜像，然后执行，打印信息。

4.5 更新docker
根据上面安装过程，重新安装即可。

### 2.2 安装包方式（更喜欢方式）

如果无法使用安装源方式（网络不通），那就可以采用安装包方式。但是每次更新都要下载最新包。

1. 下载安装包：

打开网址 *https://download.docker.com/linux/centos/7/x86_64/stable/Packages/*，下载`.rpm`合适版本下载。

> Note: To install an edge package, change the word stable in the above URL to edge. Learn about stable and edge channels.

2. 安装docker
指向包所在路径，如果是更新，把`install`改成`update`


    $ sudo yum install /path/to/package.rpm
      
      
3. 启动docker

    $ sudo systemctl start docker
    
4. 验证是否安装成功

    $ sudo docker run hello-world
    
{% asset_img a.png %}

看到红色标注部分说明安装成功。    
    
5. 更新docker-ce
下载新的安装包，用`yum -y upgrade`替换`yum -y install`,指向新的安装包。    

## 3. 卸载docker-ce

1. 卸载docker安装包：


    $ sudo yum remove docker-ce
    
2. 卸载docker安装包不会自动删除相关资源，要手动删除：


    $ sudo rm -rf /var/lib/docker
                                      
                                      
## 安装异常问题

1.安装docker遇到：package docker-ce-3:19.03.8-3.el7.x86_64 requires containerd.io >= 1.2.2-3, but none of the providers can be installed  

执行 yum install docker-ce docker-ce-cli containerd.io 提示：

_错误问题_: package docker-ce-3:19.03.8-3.el7.x86_64 requires containerd.io >= 1.2.2-3, but none of the providers can be installed

_解决方法_：   
进入阿里云镜像地址：https://mirrors.aliyun.com/docker-ce/linux/centos/7/x86_64/edge/Packages/找到你想要的或者最新的containerd.io包，拼接在阿里云地址后面，
如下：
```shell script
yum install -y https://mirrors.aliyun.com/docker-ce/linux/centos/7/x86_64/edge/Packages/containerd.io-1.2.13-3.1.el7.x86_64.rpm
```
然后再执行 yum install docker-ce docker-ce-cli containerd.io 即可。                                      
