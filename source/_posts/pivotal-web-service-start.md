---
title: 在Pivotal Web Service上发布Spring Boot应用
date: 2017-10-12 15:08:33
categories: Pivotal Web Service
tags: pws-push
---

PWS（Pivotal Web Service），由 Pivotal 公司提供的 ，可以运行Java, Grails, Play, Spring, Node.js, Ruby on Rails, Sinatra or Go 等Web应用的服务。本文将介绍一个 Hello World 级别的 Spring Boot 应用发布到 PWS 的过程。

## 1. 注册账号
在 https://run.pivotal.io/ 注册一个账号，完成手机绑定。

{% asset_img a.png %}

## 2. 安装 cf CLI

    $ wget https://s3-us-west-1.amazonaws.com/cf-cli-releases/releases/v6.29.1/cf-cli-installer_6.29.1_x86-64.rpm
    $ rpm -ivh cf-cli-installer_6.29.1_x86-64.rpm
    
其他系统安装方式：[Cloud Foundry Command Line Interface (cf CLI)](http://docs.run.pivotal.io/cf-cli/install-go-cli.html)    


## 3. 打包应用
### 3.1 下载srping-boot应用
在 Github 上克隆一个 Spring Boot 的 hello world 的项目。
`git clone https://github.com/spring-guides/gs-spring-boot.git`

### 3.2 maven打包
在 gs-spring-boot/complete 路径下执行：
`$ mvn clean package`

### 3.3 创建文件manifest.yml
gs-spring-boot/complete路径下，编写 manifest.yml 文件:
`$ vim manifest.yml`

内容如下:

    applications:
    - name: myTestApp
      path: target/gs-spring-boot-0.1.0.jar
      
说明：name 为应用程序的名字，需自定义；path 为可执行的 jar 文件路径。      

## 4. 发布应用
### 4.1 登录 CLI
`$ cf login -a api.run.pivotal.io`
账号和密码填上面注册的。

### 4.2 提交应用
`$ cf push -m 1G`

{% asset_img b.png %}

## 5.查看发布结果
1. 在 Pivotal 控制台查看发布的应用程序
{% asset_img c.png %}

2. 访问 https://mytestapp.cfapps.io/ 查看 Web 内容
{% asset_img d.png %}