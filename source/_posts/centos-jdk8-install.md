---
title: centos7下jdk8安装
date: 2017-11-12 14:32:34
categories: CentOs
tags: centos-jdk8-install
---
1、下载jdk(在官网找)
如果还没安装axel，先安装axel：> yum -y install axel

    > axel -n 10 http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.tar.gz?AuthParam=1500085329_9680fa6b22ed9ee487ee7730707b5039

2、新建安装目录

    >  mkdir /usr/lib/jvm

3、解压jdk到安装目录下

    > tar xf jdk-8u131-linux-x64.tar.gz
    > cd /usr/lib/jvm
    > mv /server/tools/jdk1.8.0_131/ /usr/lib/jvm/
    > mv jdk1.8.0_131/ jdk8 #更改名字

4、配置环境变量

    > vim /etc/profile
    
键盘按a键进入编辑模式，在末尾添加：
#jdk
export JAVA_HOME=/usr/lib/jvm/jdk8
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH

按Esc退出编辑模式
按Shift+:,然后输入wq保存退出。

5、使环境变量生效

    > source /etc/profile

6、验证安装是否成功

    > java -version

如果安装成功会看到：

    java version "1.8.0_131"
    Java(TM) SE Runtime Environment (build 1.8.0_131-b11)
    Java HotSpot(TM) 64-Bit Server VM (build 25.131-b11, mixed mode)