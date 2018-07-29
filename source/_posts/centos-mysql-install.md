---
title: centos7下安装mysql8
date: 2018-07-27 22:57:16
categories: CentOs
tags: mysql mysql8
---

本文介绍在centos7.4下源码编译安装msyql8的步骤……

参考：https://www.linuxidc.com/Linux/2018-04/152010.htm

## 源码编译安装

### 安装前清理

1. 清理旧的mysql

        rpm -pa | grep mysql
        
        yum remove mysql-xxx-xxx-
        
2. 删除旧的mysql配置文件，卸载不会自动删除。

        find / -name mysql
     
显示： 

    /etc/logrotate.d/mysql
    /etc/selinux/targeted/active/modules/100/mysql
    /etc/selinux/targeted/tmp/modules/100/mysql
    /var/lib/mysql
    /var/lib/mysql/mysql
    /usr/bin/mysql
    /usr/lib64/mysql
    /usr/local/mysql

根据需求使用以下命令 依次 对配置文件进行删除

    rm -rf /var/lib/mysql
    
3. 卸载当前系统中已安装的mariadb。centos7中默认安装，会和mysql冲突。


    rpm -qa | grep mariadb  （查找）
    
    rpm -e mysql*/mariadb*
    
    rpm -e --nodeps mysql*/mariadb*  （强制删除）
    
    ----------------------
    [root@ymu ~]# rpm -qa | grep mariadb
    mariadb-libs-5.5.56-2.el7.x86_64
    [root@ymu ~]# rpm -e mysql*/mariadb*
    error: package mysql*/mariadb* is not installed
    [root@ymu ~]# rpm -e --nodeps mysql*/mariadb*  
    error: package mysql*/mariadb* is not installed
    [root@ymu ~]# rpm -e --nodeps mariadb-libs-5.5.56-2.el7.x86_64
              

### 准备工作

1.安装依赖

    yum -y install wget  cmake gcc gcc-c++ ncurses  ncurses-devel  libaio-devel  openssl openssl-devel
        
2.下载源码包

下载网址： https://dev.mysql.com

    wget https://cdn.mysql.com//Downloads/MySQL-8.0/mysql-boost-8.0.11.tar.gz      (此版本带有boost)
     
         
3.创建mysql用户

    groupadd mysql
    useradd -r -g mysql -s /bin/false mysql
    
   
4.创建安装目录和数据目录

    mkdir -p /server/mysql
    mkdir -p /server/data/mysql
    
### 安装mysql

1.解压源码包

    [root@ymu tools]# tar -zxvf mysql-boost-8.0.11.tar.gz 
    
2.编译&安装

    [root@ymu tools]# cd mysql-8.0.11/
    [root@ymu mysql-8.0.11]# cmake .  -DCMAKE_INSTALL_PREFIX=/server/mysql \
    > -DMYSQL_DATADIR=/server/data/mysql/ \
    > -DDEFAULT_CHARSET=utf8mb4 \
    > -DDEFAULT_COLLATION=utf8mb4_general_ci \
    > -DWITH_BOOST=/server/tools/mysql-8.0.11/boost/
       
    [root@ymu mysql-8.0.11]# make && make install   
    
3.配置my.cnf文件

可能找不到该文件，如果没有，新建一个。 

          cat /etc/my.cnf
          [mysqld]
          server-id=1
          port=3306
          basedir=/server/mysql8
          datadir=/server/data/mysql8
    　　　 ##请根据实际情况添加参数
    
更改： 

    [client]
    port = 3307
    socket = /tmp/mysql.sock
    default-character-set = utf8mb4
    
    [mysqld]
    port=3307
    socket=/tmp/mysql.sock
    
    [mysql.server]
    server-id=1
    basedir=/server/mysql
    datadir=/server/data/mysql
    pid-file=/server/data/mysql/mysql.pid
    
    character-set-server = utf8mb4
    
    slow_query_log=1
    long_query_time=5
    slow_query_log_file=/server/data/mysql/mysql-slow.log
    
    default_storage_engine=InnoDB
    innodb_file_per_table = 1
    innodb_open_files = 500
    innodb_buffer_pool_size = 64M
    innodb_write_io_threads = 4
    innodb_read_io_threads = 4
    innodb_thread_concurrency = 0
    innodb_purge_threads = 1
    innodb_flush_log_at_trx_commit = 2
    innodb_log_buffer_size = 2M
    innodb_log_file_size = 32M
    innodb_log_files_in_group = 3
    innodb_max_dirty_pages_pct = 90
    innodb_lock_wait_timeout = 120

    

参考：

    [client]
    port = 3307
    socket = ~/tmp/mysql.sock
    default-character-set = utf8mb4
    
    [mysqld]
    port = 3307
    socket = /home/mutian/tmp/mysql.sock
    
    [mysql.server]
    basedir = /home/mutian/dev/tools/mysql
    datadir = /home/mutian/dev/data/mysql
    pid-file = /home/mutian/dev/data/mysql/mysql.pid
    user = mysql
    bind-address = 0.0.0.0
    server-id = 1
    
    init-connect = 'SET NAMES utf8mb4'
    character-set-server = utf8mb4
    
    #skip-name-resolve
    #skip-networking
    back_log = 300
    
    max_connections = 1000
    max_connect_errors = 6000
    open_files_limit = 65535
    table_open_cache = 128
    max_allowed_packet = 4M
    binlog_cache_size = 1M
    max_heap_table_size = 8M
    tmp_table_size = 16M
    
    read_buffer_size = 2M
    read_rnd_buffer_size = 8M
    sort_buffer_size = 8M
    join_buffer_size = 8M
    key_buffer_size = 4M
    
    thread_cache_size = 8
    
    query_cache_type = 1
    query_cache_size = 8M
    query_cache_limit = 2M
    
    ft_min_word_len = 4
    
    log_bin = mysql-bin
    binlog_format = mixed
    expire_logs_days = 30
    
    slow_query_log = 1
    long_query_time = 1
    slow_query_log_file = /home/mutian/dev/data/mysql/mysql-slow.log
    
    performance_schema = 0
    explicit_defaults_for_timestamp
    
    #lower_case_table_names = 1
    
    skip-external-locking
    
    default_storage_engine = InnoDB
    #default-storage-engine = MyISAM
    innodb_file_per_table = 1
    innodb_open_files = 500
    innodb_buffer_pool_size = 64M
    innodb_write_io_threads = 4
    innodb_read_io_threads = 4
    innodb_thread_concurrency = 0
    innodb_purge_threads = 1
    innodb_flush_log_at_trx_commit = 2
    innodb_log_buffer_size = 2M
    innodb_log_file_size = 32M
    innodb_log_files_in_group = 3
    innodb_max_dirty_pages_pct = 90
    innodb_lock_wait_timeout = 120
    
    bulk_insert_buffer_size = 8M
    myisam_sort_buffer_size = 8M
    myisam_max_sort_file_size = 10G
    myisam_repair_threads = 1
    
    interactive_timeout = 28800
    wait_timeout = 28800
    
    [mysqldump]
    quick
    max_allowed_packet = 16M
    
    [myisamchk]
    key_buffer_size = 8M
    sort_buffer_size = 8M
    read_buffer = 4M
    write_buffer = 4M

4.目录权限修改

    chown -R mysql:mysql /server/mysql
    chown -R mysql:mysql /server/data/mysql
    chmod 755 /server/mysql -R
    chmod 755 /server/data/mysql -R

5.初始化

    bin/mysqld --initialize --user=mysql
    bin/mysql_ssl_rsa_setup

6.启动mysql

    bin/mysqld_safe --user=mysql &

7.修改账号密码

- 如果出现错误：

       ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2)
       
说明服务没启动成功。     

- 错误： 

       [root@ymu mysql]# bin/mysql  -uroot -p
       Enter password: 
       ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)

空密码无法登录。

处理（修改密码）： 

    bin/mysqld_safe --user=mysql --skip-grant-tables &
     
    mysql -uroot -p     //密码，直接按回车登录
    use mysql;
    select host, user, authentication_string, plugin from user; 
    update user set authentication_string='' where user='root';
    flush privileges;
    //更更改密码
    ALTER user 'root'@'localhost' IDENTIFIED BY 'ymu123@';
    quit;
    
    ------------------------------
    如果执行
    ALTER user 'root'@'localhost' IDENTIFIED BY 'ymu123@';
    报错误。
    
    按下面处理，执行：flush privileges;然后在执行更改密码语句就ok了：
    mysql> ALTER user 'root'@'localhost' IDENTIFIED BY 'ymu123@';
    ERROR 1290 (HY000): The MySQL server is running with the --skip-grant-tables option so it cannot execute this statement
    mysql> 
    mysql> flush privileges;
    Query OK, 0 rows affected (0.01 sec)
    
    mysql> ALTER user 'root'@'localhost' IDENTIFIED BY 'ymu123@';
    Query OK, 0 rows affected (0.06 sec)
    
    mysql>   
    
    如果报错如下信息：
    Error: Cannot retrieve repository metadata (repomd.xml) for repository: InstallMedia. Please verify its path and try again
     You could try using --skip-broken to work around the problem
     You could try running: rpm -Va --nofiles --nodigest
    
    我们只要到/etc/yum.repo.s下面把packetxxxx.repo和redhat.repo两个文件删除掉，再启动就可以了， 
    
    ------------------------------------------
    查看密码是否已经重置：
    mysql> select host, user, authentication_string, plugin from user;
    +-----------+------------------+------------------------------------------------------------------------+-----------------------+
    | host      | user             | authentication_string                                                  | plugin                |
    +-----------+------------------+------------------------------------------------------------------------+-----------------------+
    | localhost | mysql.infoschema | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE                              | mysql_native_password |
    | localhost | mysql.session    | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE                              | mysql_native_password |
    | localhost | mysql.sys        | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE                              | mysql_native_password |
    | localhost | root             | $A$005$ }DQPo%':5'_#-QhPn/ZULDCRuo2Xh8w2uhS1.zZ/U.W8d7zWwlmpTB3D5 | caching_sha2_password |
    +-----------+------------------+------------------------------------------------------------------------+-----------------------+
    4 rows in set (0.00 sec)
    
    可以看到root用户的密码已经更改。然后重启mysql登录试试。
    
    --------------------------
    客户端连接报错：客户端连接caching-sha2-password问题。
    这是因为msyql8对密码加密的规则导致，navicat不支持。所以，需要更改加密规则：
    
    在服务器，通过mysql客户端登入：
    [root@ymu ~]# mysql -uroot -p
    
    #修改加密规则  
    ALTER USER 'root'@'localhost' IDENTIFIED BY 'password' PASSWORD EXPIRE NEVER; 
    或者：
    ALTER USER 'root'@'%' IDENTIFIED BY 'password' PASSWORD EXPIRE NEVER; 
    
    #更新密码（mysql_native_password模式）    
    ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '{NewPassword}';
    或者：
    ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '{NewPassword}';

    实际操作过程：
    mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'ymu123@' PASSWORD EXPIRE NEVER;
    ERROR 1396 (HY000): Operation ALTER USER failed for 'root'@'localhost'
    mysql> ALTER USER 'root'@'%' IDENTIFIED BY 'password' PASSWORD EXPIRE NEVER;
    Query OK, 0 rows affected (0.05 sec)
    
    mysql> ALTER USER 'root'@'*' IDENTIFIED WITH mysql_native_password BY 'ymu123@';
    ERROR 1396 (HY000): Operation ALTER USER failed for 'root'@'*'
    mysql> ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'ymu123@';
    Query OK, 0 rows affected (0.10 sec)

    
设置密码不能是`123456`这些简单的密码，会通不过。所以设置复杂点`ymu123@`。     
     
表格中有以下信息：  
host: 允许用户登录的 ip ‘位置’ % 表示可以远程；  
user: 当前数据库的用户名；    
authentication_string: 用户密码（在mysql 5.7.9以后废弃了password字段和password()函数）；  
plugin： 密码加密方式；  

然后重启就可以登录了。 

参考： https://blog.csdn.net/xinpengfei521/article/details/80400142    
     
8.创建软链接（非必要）

    ln -s /server/mysql/bin/* /usr/local/bin/

9.添加到启动（非必要）

开启自动启动mysql：

    cp support-files/mysql.server /etc/init.d/mysql.server
    chmod +x /etc/init.d/mysql.server
    chkconfig --add mysql.server
    chkconfig mysql.server on
    
    ------ 参考来源 ----
    /bin/cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
    chmod +x /etc/init.d/mysqld
    chkconfig --add mysqld
    chkconfig mysqld on
    
10.查看启动状态、启动、停止、重启

`mysql.server`对应上面设置自启动的：`/etc/init.d/mysql.server`

查看状态：

`systemctl status mysql.server.service`  

或者：
`service mysql.server status`

停止：`service mysql.server stop`

启动：`service mysql.server start`

重新启动：`service mysql.server reload`

  
    
## 二进制安装包rpm安装（推荐）    
                  