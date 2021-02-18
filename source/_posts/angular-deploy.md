---
title: 部署angular应用到生产环境
date: 2021-01-16 09:45:30
categories: angular
tags:
---

当你准备把 Angular 应用部署到远程服务器上时，有很多可选的部署方式。下面介绍在nginx环境下部署，还有制作docker镜像方式部署。

## 构建，并部署到nginx服务器

#### 构建应用生产环境

1.使用开发环境进行构建

```shell script
ng build --prod
```

2.把输出目录（默认为 dist/）下的每个文件都复制到到服务器上的某个目录下。

3.配置服务器，让缺失的文件都重定向到 index.html 上。

#### nginx配置文件

使用 try_files 指向 index.html

```text
try_files $uri $uri/ /index.html;
```

完整配置：

```text
location / {
    root   /usr/share/nginx/html;
    index  index.html index.htm login.html;
    try_files $uri $uri/ /index.html;
}
```

## 制作nginx docker镜像整体发布