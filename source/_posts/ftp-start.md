---
title: ftp入门
date: 2018-12-27 09:30:53
categories: 文件存储系统
tags:
---

## CentOs7下搭建ftp服务器

### 方式一：docker下安装

1.拉取镜像：

    docker pull fauria/vsftpd
    
2.启动容器：

    docker run -d -v /dockers/vsftpd:/home/vsftpd \
    -p 20:20 -p 21:21 -p 21100-21110:21100-21110 \
    -e FTP_USER=admin -e FTP_PASS=123456 \
    -e PASV_ADDRESS=127.0.0.1 -e PASV_MIN_PORT=21100 -e PASV_MAX_PORT=21110 \
    --name vsftpd --restart=always fauria/vsftpd   
    
    ------------------------------------
    docker logs vsftpd