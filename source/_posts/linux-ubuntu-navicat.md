---
title: ubuntu下使用数据库客户端软件navicat
date: 2018-08-10 10:02:17
categories: ubuntu
tags: ubuntu-navicat
---

本文记录在ubuntu系统下使用navicat的各种问题……

网址：http://navicat.com.cn/

## 安装

直接打开官网下载即可。

1.下载好，解压，并进入文件根目录。会看到`start_navicat`文件。   

2.启动

把`export LANG="en_US.UTF-8"`这句改为`export LANG="zh_CH.UTF-8"`。    

执行：`./start_navicat`。即可启动。会下载`wine`。   

3.解决乱码

- 给系统安装字体`sudo apt install fonts-wqy-zenhei`。   
- 启动navicat，点解倒数第三个菜单，选择最后一个选项(options),在第一个tab下，下拉，选择最后一种字体`WenQuanYi Zen Hei Sharp`。  
- 退出应用，然后重新启动，应该可以看到正确的显示了。

## 破解

进入目录`cd ~/.navicat64`,删除目录下的system.reg文件即可。如果还是不行，可以把整个.navicat64目录删除。然后重启。

## 中文乱码

下载最新中文版本。

第一次进入主界面，乱码。    

点击菜单：工具->选项->常规->界面字体, 下拉选择：AR PL UMing CN.然后关闭，再次启动navicat即可。