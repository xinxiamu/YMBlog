---
title: aws云服务器使用
date: 2018-06-21 14:22:55
categories: aws
tags: aws云服务器
---

本文记录在使用亚马逊云服务器aws过程中遇到的一些问题……

## 登录到aws云服务器

- 下载私钥`*.pem`,放到指定目录下。
-  给予私钥权限。`chmod 400 .pem`
- ssh登录。`ssh -i ~/aws-server-key/Seoul-Server-Key.pem ubuntu@13.209.47.139`
