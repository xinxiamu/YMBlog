---
title: idea 使用问题收集
date: 2017-11-10 09:52:52
categories: idea
tags:
---

## maven控制台输出乱码

问题描述：

执行maven命令，控制台输出中文乱码。

解决方案：

解决：maven默认环境为jdk,只需要改如下即可：
在IDEA中，打开File | Settings | Build, Execution, Deployment | Build Tools | Maven | Runner在VM Options中
添加-Dfile.encoding=GBK，切记一定是GBK。即使用UTF-8的话，依然是乱码，这是因为Maven的默认平台编码是GBK，
如果你在命令行中输入mvn -version的话，会得到如下信息，根据Default locale可以看出