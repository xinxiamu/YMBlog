---
title: mysql使用问题记录
date: 2018-06-01 22:46:53
categories: mysql
tags: mysql使用问题收藏
---

本文记录在实际使用mysql过程遇到的问题……

## mysql5.7更改数据库密码强度和长度

    set global validate_password_policy=0;  
    set global validate_password_length=4;  

## mysql5.7，创建登录用户，并对用户赋权只能查看某个数据库

    CREATE USER 'redmine'@'localhost' IDENTIFIED BY 'my_password';
    GRANT ALL PRIVILEGES ON redmine.* TO 'redmine'@'localhost';
    flush privileges;
    
 创建用户redmine,密码为：redmine，只对数据库redmine拥有读写权限。   