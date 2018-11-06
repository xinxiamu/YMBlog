---
title: aws云服务器使用
date: 2018-06-21 14:22:55
categories: aws
tags: aws云服务器
---

本文记录在使用亚马逊云服务器aws过程中遇到的一些问题……

### 1.登录到aws云服务器

- 下载私钥`*.pem`,放到指定目录下。
-  给予私钥权限。`chmod 400 .pem`
- ssh登录。`ssh -i ~/aws-server-key/Seoul-Server-Key.pem ubuntu@13.209.47.139`

### 2.在安全组开放端口

### 3.上传文件aws服务器

- IPv4：

        # chmod 400 /path/my-key-pair.pem
        # scp -i /path/my-key-pair.pem /path/SampleFile.txt ec2-user@ec2-198-51-100-1.compute-1.amazonaws.com:~
        
- (仅限 IPv6) 或者,您可以使用实例的 IPv6 地址传输文件。IPv6 地址必须用方括号 ([]) 括起,方括号必
  须转义 (\)。
  
        scp -i /path/my-key-pair.pem /path/SampleFile.txt ec2-user@
        \[2001:db8:1234:1a00:9691:9503:25ad:1761\]:~

                

_小提示_:  
对于 Amazon Linux,用户名为 ec2-user。对于 RHEL,用户名称是 ec2-user 或 root。
对于 Ubuntu,用户名称是 ubuntu 或 root。对于 Centos,用户名称是 centos。对于
Fedora,用户名称是 ec2-user。对于 SUSE,用户名称是 ec2-user 或 root。另外,如果
ec2-user 和 root 无法使用,请与您的 AMI 供应商核实。

### 4.下载文件到本机

- IPv4：

        scp -i /path/my-key-pair.pem ec2-user@ec2-198-51-100-1.compute-1.amazonaws.com:~/
        SampleFile.txt ~/SampleFile2.txt


- (仅限 IPv6) 或者,您可以使用实例的 IPv6 地址反方向传输文件。

        scp -i /path/my-key-pair.pem ec2-user@\[2001:db8:1234:1a00:9691:9503:25ad:1761\]:~/
        SampleFile.txt ~/SampleFile2.txt
