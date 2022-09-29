---
title: ngrok使用
date: 2018-08-13 15:36:39
categories: ngrok
tags: ngrok使用
---

官网：https://ngrok.com

## 常用问题

#### 后台运行

ngrok 用 & 不能后台运行

这就要使用screen这个命令

首先安装screen

apt-get install screen

之后运行

screen -S 任意名字（例如：keepngork）

然后运行ngrok启动命令

最后按快捷键

ctrl+A+D
