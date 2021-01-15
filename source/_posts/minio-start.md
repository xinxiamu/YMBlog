---
title: 分布式文件存储系统 MINIO入门
date: 2020-06-18 11:05:36
categories: 文件存储系统
tags:
---

发现一个比FastDFS更加简单、体验更加好的文件存储系统，撸它……

## 安装-Docker环境下

```shell script
docker run -p 9000:9000 \
--restart=always \
--name minio1 \
-v /mnt/data:/data \
-e "MINIO_ROOT_USER=izajzo6yycy5d65vexjmdvtts" \
-e "MINIO_ROOT_PASSWORD=kvb9fv35c14soranzsz8bwzyf" \
-d minio/minio server /data
```


## Java客户端使用

### 上传文件

### 查看文件

## 集成nginx访问文件

## 参考资源

[官网](https://min.io/)
