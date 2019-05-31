---
title: mysql使用问题记录
date: 2018-06-01 22:46:53
categories: mysql
tags: mysql使用问题收藏
---

本文记录在实际使用mysql过程遇到的问题……

## 配置my.cnf

    [mysqld]
    port = 3910
    character-set-client-handshake = FALSE
    character-set-server = utf8mb4
    collation-server = utf8mb4_unicode_ci
    init_connect='SET NAMES utf8mb4'
    default_authentication_plugin = mysql_native_password
    skip-name-resolve
    
    log-bin = mysql-bin
    server-id = 1
    
    [client]
    default-character-set=utf8mb4
    
    [mysql]
    default-character-set=utf8mb4

## mysql5.7更改数据库密码强度和长度

    set global validate_password_policy=0;  
    set global validate_password_length=4;  

## mysql5.7，创建登录用户，并对用户赋权只能查看某个数据库

    CREATE USER 'redmine'@'localhost' IDENTIFIED BY 'my_password';
    GRANT ALL PRIVILEGES ON redmine.* TO 'redmine'@'localhost';
    flush privileges;
    
 创建用户redmine,密码为：redmine，只对数据库redmine拥有读写权限。   
 
 ## 设置超时
 
 数据库连接时间太短，会自动释放，业务代码没执行完成，就断开了。设置太长，会导致整个系统被拖慢。所以要设置一个恰当的时间。
 
     SHOW GLOBAL VARIABLES LIKE '%timeout%'
     
     SET GLOBAL wait_timeout=10000
     
     