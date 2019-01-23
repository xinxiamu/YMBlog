---
title: mysql复制
date: 2018-10-24 22:00:17
categories: mysql
tags: mysql一主多从配置
---

系统性能的提升，高并发的实现，其中一个部分就是提高数据库性能。而数据库的读写分离，就是其中一个优化。在主库写入，在读库做查询，减少主库的请求压力，这对整个系统将会有大大性能提升。   
但是，这里会出现一个问题，就是主从库在数据同步时候，会出现迟延问题，如何保证数据同步及时，保证数据一致性将是个难题。但是，在一般要求不是实时的系统，是没问题。     
下面我们就来逐个探索这些问题的……   

## docker环境搭建mysql关系数据库一主多从架构

docker安装mysql。参考章节{% post_link  docker-app-install docker安装mysql %}  

### 在三台服务器安装mysql服务器。    

其中一台作为master，一台slave1，一台slave2，一主二从配置。

1.修改每台mysql服务配置：    

- 主库master配置修改：   


    [mysqld] 
    port = 3910
    character-set-client-handshake = FALSE 
    character-set-server = utf8mb4 
    collation-server = utf8mb4_unicode_ci
    init_connect='SET NAMES utf8mb4'
    default_authentication_plugin = mysql_native_password
    
    log-bin = mysql-bin
    server-id = 1
    
    [client]
    default-character-set=utf8mb4
    
    [mysql]
    default-character-set=utf8mb4
    
_注_： 主要就是在原来配置上，添加下面两行：  

    log-bin = mysql-bin
    server-id = 1

> _说明_:   
server-id=1： 唯一服务器ID，非0整数，不能和其他服务器的server-id重复
log-bin=mysql-bin：开启二进制日志功能。使用binary logging，mysql-bin是log文件名的前缀

- 从库slave1，配置文件修改：  


    [mysqld] 
    port = 3911
    character-set-client-handshake = FALSE 
    character-set-server = utf8mb4 
    collation-server = utf8mb4_unicode_ci
    init_connect='SET NAMES utf8mb4'
    default_authentication_plugin = mysql_native_password
    
    log-bin = mysql-bin
    server-id = 2
    
    [client]
    default-character-set=utf8mb4
    
    [mysql]
    default-character-set=utf8mb4
    
- 从库slave2，配置文件修改：  


    [mysqld] 
    port = 3912
    character-set-client-handshake = FALSE 
    character-set-server = utf8mb4 
    collation-server = utf8mb4_unicode_ci
    init_connect='SET NAMES utf8mb4'
    default_authentication_plugin = mysql_native_password
    
    log-bin = mysql-bin
    server-id = 3
    
    [client]
    default-character-set=utf8mb4
    
    [mysql]
    default-character-set=utf8mb4
    
2.重新启动三个mysql服务：    

    docker restart mysql容器  
    
3.用navicat工具连接数据库

保证三台mysql数据都能正确连接。   

{%asset_img a-1.png%} 

### 配置主从关系

为了方便直接在navicat客户端连接操作。  

#### 在master操作   

1.新增用来复制数据的用户：  

    CREATE USER 'slave'@'%' IDENTIFIED BY '123456';
    
用户名：slave  密码：123456

2.对用户slave授权：   

    GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'slave'@'%';
    
3.查看授权： 

    show grants for 'slave'@'%';
    
4.查看主库状态：   

    show master status;
    
{%asset_img a-2.png%}  

如图，字段File，Position,后面会用到。   

到此，不要再操作主库。 否则这连个字段会变。   

#### 从库slave1操作  

1.连接从库slave1，执行如下命令：    

    change master to master_host='192.168.199.101', master_user='slave', master_password='123456', master_port=3910, master_log_file='mysql-bin.000001', master_log_pos= 155, master_connect_retry=30;
    
_命令说明：_ 

- master_host： 主数据库ip地址，navicat连接的master库的ip地址即可。   
- master_user： 上面在主库创建的用来复制数据的用户名。
- master_port： 主库的端口，navicat连接的端口。  
- master_password： 用于同步数据的主库用户的对应密码。    
- master_log_file: 指定 Slave 从哪个日志文件开始复制数据，即上文中提到的 File 字段的值。    
- master_log_pos： 从哪个 Position 开始读，即上文中提到的 Position 字段的值。
- master_connect_retry： 如果连接失败，重试的时间间隔，单位是秒，默认是60秒。

2.查看从库状态：   

    show slave status;

正常会看到，Slave_IO_State字段是空的，Slave_IO_Runngin为NO，Slave_SQL_Running为No。 
那是因为还没启动主从复制服务。 

3.启动主从复制功能： 

    start slave;
    
启动后，再次查看从库状态：   

    show slave status;
    
{%asset_img a-3.png%}    
        
如果看到： 
                   
Slave_IO_State：  Waiting for master to send event       
Slave_IO_Runngin： YES       
Slave_SQL_Running: YES  

则说明主从配置成功了。 

4.停止主从配置服务： 

    stop slave;
    
5.刷新主从配置信息，重新设置主从配置：    

    reset slave all;    

#### 从库slave2操作

从库slave2的操作，和在从库slave1上的操作一样。

    show slave status;
    
    change master to master_host='192.168.199.101', master_user='slave', master_password='123456', master_port=3910, master_log_file='mysql-bin.000001', master_log_pos= 155, master_connect_retry=30;
    
    start slave;
    
    show slave status;

### 测试主从配置
 
在主库，随便新建数据库，新增表，然后打开两个从库，你会马上看到同样的库和表。  

恭喜你，一主二从配置成功了。

## 双主双从配置

待续……


      