---
title: docker环境安装redmine
date: 2020-09-02 11:16:44
categories: redmine
tags:
---

本文介绍在docker环境下安装Redmine项目开发管理软件系统……

## 安装mysql

```shell script
docker run --name mysql-redmine -p 3309:3306 --restart always --privileged=true -v /redmine/db/conf:/etc/mysql/conf.d -v /redmine/db/logs:/logs -v /redmine/db/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=a1234567@! -e MYSQL_USER=redmine -e MYSQL_PASSWORD=a1234567@! -e MYSQL_DATABASE=redmine -d mysql:5.7
```


## 安装redmine

```shell script
docker run -d --name redmine -p 8889:3000 --restart always -v /redmine/redmine-dir/files:/usr/src/redmine/files -e REDMINE_DB_MYSQL=mysql-redmine -e REDMINE_DB_USERNAME=redmine -e REDMINE_DB_PASSWORD=a1234567@! --link mysql-redmine:mysql redmine
```

## 迁移redmine系统数据

#### 数据库迁移


#### 文件迁移


#### 插件

