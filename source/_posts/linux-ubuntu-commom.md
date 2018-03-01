---
title: ubuntu使用常用命令收集
date: 2018-03-01 16:02:21
categories: ubuntu
tags:
---

记录在使用ubuntu系统过程中常见命令……

## 查看占用端口

`netstat -ln|grep 8388`或者`lsof -i:8388`

## 关闭端口下应用

`kill -9 PID号`


## 防火墙

1.安装

`sudo apt-get install ufw` 

2.启用

    sudo ufw enable
    sudo ufw default deny
    运行以上两条命令后，开启了防火墙，并在系统启动时自动开启。

3.开启/禁用

    sudo ufw allow|deny [service]
    打开或关闭某个端口，例如：
    sudo ufw allow smtp　允许所有的外部IP访问本机的25/tcp (smtp)端口
    sudo ufw allow 22/tcp 允许所有的外部IP访问本机的22/tcp (ssh)端口
    sudo ufw allow 53 允许外部访问53端口(tcp/udp)
    sudo ufw allow from 192.168.1.100 允许此IP访问所有的本机端口
    sudo ufw allow proto udp 192.168.0.1 port 53 to 192.168.0.2 port 53
    sudo ufw deny smtp 禁止外部访问smtp服务
    sudo ufw delete allow smtp 删除上面建立的某条规则
    
4.查看防火墙状态

`sudo ufw status`    

开启/关闭防火墙 (默认设置是’disable’)

`# ufw enable|disable`

5.UFW 使用范例：

    允许 53 端口
    
    $ sudo ufw allow 53
    
    禁用 53 端口
    
    $ sudo ufw delete allow 53
    
    允许 80 端口
    
    $ sudo ufw allow 80/tcp
    
    禁用 80 端口
    
    $ sudo ufw delete allow 80/tcp
    
    允许 smtp 端口
    
    $ sudo ufw allow smtp
    
    删除 smtp 端口的许可
    
    $ sudo ufw delete allow smtp
    
    允许某特定 IP
    
    $ sudo ufw allow from 192.168.254.254
    
    删除上面的规则
    
    $ sudo ufw delete allow from 192.168.254.254