---
title: jenkins-java-maven持续集成（CentOs7）
date: 2017-11-28 09:15:26
categories: 持续集成
tags: jenkins-java-maven
---

传统的开发是开发，打包测试，开发，打包测试，且每次都是全量打包。这给运维，测试带来大量没必要的工作量，同时每次全量打包导致系统每次测试不全面，bug不断，测试，开发，运维叫苦连天。因此持续集成开发势在必行。

## 1. 安装并运行jenkins-war
- [下载](https://jenkins.io/download/)
- 运行 `java -jar jenkins.war --httpPort=8080.`
- 在浏览器访问 http://localhost:8080. 记得开防火墙。

图一：
{% asset_img setup-jenkins-01-unlock-jenkins-page.jpg %}

## 2. 初次访问配置
- 按图一红色提示，在服务器对应目录下找到安全密码，拷贝进去登录。
- 按照页面想到，安装插件，如果不确定要安装什么插件，那就选择推荐的插件按钮即可。
- 创建管理员账号，按照页面设置即可。

这样，下面就可以开始使用jenkins了。


## 3. maven下持续构建java应用
- 安装maven环境。[下载maven](http://mirror.bit.edu.cn/apache/maven/maven-3/3.5.2/binaries/apache-maven-3.5.2-bin.tar.gz)

- 

