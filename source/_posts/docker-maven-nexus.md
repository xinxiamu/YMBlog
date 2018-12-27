---
title: 使用maven插件构建docker镜像并注册到nexus私仓
date: 2018-12-27 14:29:17
categories: docker
tags:
---

Maven是一个强大的项目管理和构建工具，下面我们介绍利用Maven插件构建Docker镜像并注册到Nexus私仓里面，这样，在服务器或者其它地方就可以直接拉取镜像并运行，这对服务共享或者服务器部署应用起到极大的方便。      
在maven中央仓库，可以搜索到好几个docker-maven-pluging。

下面我们采用一款由Spotify公司开发的Maven插件[docker-maven-plugin](https://github.com/spotify/docker-maven-plugin)。


