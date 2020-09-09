---
title: docker环境安装redmine
date: 2020-09-02 11:16:44
categories: redmine
tags:
---

本文介绍在docker环境下安装Redmine项目开发管理软件系统……

## 安装mysql

```shell script
docker run --name mysql-redmine -p 3309:3306 --restart always --privileged=true -v /server/redmine/db/conf:/etc/mysql/conf.d -v /server/redmine/db/logs:/logs -v /server/redmine/db/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=a1234567@! -e MYSQL_USER=redmine -e MYSQL_PASSWORD=a1234567@! -e MYSQL_DATABASE=redmine -d mysql:5.7
```


## 安装redmine

```shell script
docker run -d --name xc-redmine -p 8888:3000 --restart always -v /server/redmine/redmine-dir/files:/usr/src/redmine/files:z -v /server/redmine/redmine-dir/plugins:/usr/src/redmine/plugins:z -e REDMINE_DB_MYSQL=mysql-redmine -e REDMINE_DB_USERNAME=redmine -e REDMINE_DB_PASSWORD=a1234567@! --link mysql-redmine:mysql redmine
```

## docker-compose安装

```shell script
version: '2.0'
 
services:
 
  redmine:
    image: redmine
    restart: always
    ports:
      - 3000:3000
    environment:
      REDMINE_DB_MYSQL: db
      REDMINE_DB_PASSWORD: root123
    volumes:
      - ./redmine/data:/usr/src/redmine/files:z
      - ./redmine/plugins:/usr/src/redmine/plugins:z
 
  db:
    image: mysql:5.7
    restart: always
    volumes:
      - ./mysql/data:/var/lib/mysql:z      
    environment:
      MYSQL_ROOT_PASSWORD: root123
      MYSQL_DATABASE: redmine
    ports:
      - 3306:3306
```

## 迁移redmine系统数据

#### 数据库迁移

直接拷贝同步以前数据库到最新的库即可。

#### 文件迁移

进入以前的redmine安装目录，拷贝整个files文件夹到新的filse文件夹路径下，文件就能访问了。

#### 插件

进入redmine容器，按照之前介绍方式安装插件。`docker exec -it redmine /bin/bash`。

或者：

移除或者拷贝插件到plugins映射卷下，并`docker restart redmine`即可实现插件的安装卸载。
