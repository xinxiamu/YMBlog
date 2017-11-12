---
title: centos7下搭建maven私仓Nexus
date: 2017-11-12 14:47:38
categories: CentOs
tags: centos-nexus
---

项目组件化，共用jar包，统一获取墙外jar包……

## 安装jdk
参考其他文章……


## 下载Nexus Repository OSS
> 下载地址：http://www.sonatype.com/download-oss-sonatype

    [root@iZj6ca50pk1lwxqo14jss8Z nexus]# axel -n 10 https://sonatype-download.global.ssl.fastly.net/nexus/3/nexus-3.6.0-02-unix.tar.gz

解压：
    
    [root@iZj6ca50pk1lwxqo14jss8Z nexus]# tar -zxvf nexus-3.6.0-02-unix.tar.gz
    
## 启动
    
    [root@iZj6ca50pk1lwxqo14jss8Z nexus]# cd nexus-3.6.0-02/bin/
    [root@iZj6ca50pk1lwxqo14jss8Z bin]# ./nexus run

如果启动成功，可以看到：
 > Started Sonatype Nexus OSS 3.6.0-02

防火墙开启8081端口。注意还要在阿里云控制后台安全组开启端口。

在浏览器访问：http://47.52.236.72:8081/，可以看到：

{% asset_img a.png %}

## 配置为Linux Service

1. 编辑`bin/nexus.rc`：

`[root@iZj6ca50pk1lwxqo14jss8Z ~]# vim /server/java/nexus/nexus-3.6.0-02/bin/nexus.rc`

添加： 
> run_as_user="root"

2. 在`/etc/init.d`放nexus软连接

` ln -s /server/java/nexus/nexus-3.6.0-02/bin/nexus /etc/init.d/nexus`

3. 设置服务随系统自启
命令：*chkconfig*


    [root@iZj6ca50pk1lwxqo14jss8Z ~]# cd /etc/init.d/
    [root@iZj6ca50pk1lwxqo14jss8Z init.d]# chkconfig nexus on

5. 启动


    [root@iZj6ca50pk1lwxqo14jss8Z ~]# service nexus start
    WARNING: ************************************************************
    WARNING: Detected execution as "root" user.  This is NOT recommended!
    WARNING: ************************************************************
    Starting nexus

## 界面操作

*1. 登录*

默认账号密码：
username: admin
pwd: admin123

{% asset_img b.png %} 

*2. 修改admin密码*

{% asset_img c.png %}

点击`More`

{% asset_img d.png %}
    
## 创建maven仓库


    
