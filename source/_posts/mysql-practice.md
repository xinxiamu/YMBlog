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
    # 一下三个配置，解决too manay connection问题。
    max_connections=1000 # 最大连接数，默认100
    wait_timeout = 300 #
    interactive_timeout = 500
    
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
     
## mysql too many connections 解决方法     

以下三个配置，解决too manay connection问题。

    max_connections=1000 # 最大连接数，默认100
    wait_timeout = 300 #
    interactive_timeout = 500
    
## 新建用户并授权
只能本机登录：
```shell script
CREATE USER 'service_scf_root'@'localhost' IDENTIFIED BY '84012d469d52d8e29a4b95ef87dd6d97';

GRANT ALL PRIVILEGES ON service_scf.* TO'service_scf_root'@'localhost';

flush privileges;
```    

可远程登录：
```shell script
CREATE USER 'service_scf_root'@'%' IDENTIFIED BY '84012d469d52d8e29a4b95ef87dd6d97';

GRANT ALL PRIVILEGES ON service_scf.* TO 'service_scf_root'@'%';

flush privileges;
```
    
参考：
https://www.jianshu.com/p/fc40067c4dc9      
https://jingyan.baidu.com/article/fc07f989c5c6bd52fee5192c.html 


1.分组查询报错

错误：

> 1055 - Expression #1 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'service_abs_ss_local.hta.id' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by

解决方案：

参考：

https://blog.csdn.net/qq_42175986/article/details/82384160

## 创建用户，授权

```shell script
CREATE USER 'jinnuo_root'@'%' IDENTIFIED BY 'a1234567';
grant select,insert,update,references,delete,create,drop,alter,index,trigger,create routine,alter routine,execute,create view,show view,lock tables,event on jinnuo.*  to  jinnuo_root;
flush  privileges;
ALTER USER 'jinnuo_root'@'%' IDENTIFIED WITH mysql_native_password BY 'a1234567';
flush privileges;
```

## 连接错误Public Key Retrieval is not allowed（解决方式）

在url连接添加属性`allowPublicKeyRetrieval=true



