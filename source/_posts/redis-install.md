---
title: redis安装
date: 2018-01-18 20:55:18
categories: redis
tags: redis-install
---

本文介绍在linux系统下redis的安装使用……

## redis在ubuntu系统下的安装

### 一、下载，安装，测试

Download, extract and compile Redis with:

    $ wget http://download.redis.io/releases/redis-3.2.0.tar.gz
    $ tar xzf redis-3.2.0.tar.gz
    $ cd redis-3.2.0
    $ make

The binaries that are now compiled are available in the src directory. Run Redis with:

    $ src/redis-server

You can interact with Redis using the built-in client:

    $ src/redis-cli redis> set foo bar OK redis> get foo "bar"

### 二、设置直接使用redis的启动，客户端命令
make编译redis后，执行命令

       $ cd redis-3.2.0
       $ sudo make install
       
src中的目录会被复制到/usr/local/bin，这样直接就可以使用src下的一些执行命令。然后可以直接在命令行下执行这些redis命令：

    xiaocao@xiaocao-pc:~$ redis-cli
     127.0.0.1:6379> get foo
     "123456"
     127.0.0.1:6379>
启动服务：`xiaocao@xiaocao-pc:~$ redis-server`

### 三、src目录下执行文件说明：

redis-check-aof //AOF文件修复工具
redis-cli //Redis命令行客户端，最常用
redis-server //Redis服务器，最常用
redis-benchmark //Redis性能测试工具
redis-check-rdb //
redis-sentinel     //Sentinel服务器，2.8版本后

### 四、启动，停止redis服务

1、直接启动：make install后
执行命令：`$ redis-server`     //默认端口是6379
自定义端口启动：
执行命令：`$ redis-server --port 6380`
2、通过初始化脚本启动redis，使得redis能随系统自动运行（在生产环境服务器更推荐此方法）
(1)配置初始化脚本。
>a. `$ cd ~/java/redis-3.2.0/utils`
将utils目录下的redis_init_script文件复制一份到/etc/init.d目录中。文件名改为redis_端口号。端口号是redis监听的端口号，
客户端连接的端口号。

>b.修改（redis_端口号） 脚本第6行的REDISPORT变量的值为同样的端口号。

（2）建立需要的文件夹：
>/etc/redis      //存放Redis的配置文件
/var/redis/端口号     //存放Redis的持久化文件

(3)修改配置文件：

    a、$ cd  ~/java/redis-3.2.0
    b、将配置文件模板redis.conf复制一份到/etc/redis目录中，以端口号命名，如“6380.conf”。
    c、修改参数( 6380.conf )：
    参数：daemonize       值： yes            说明：使redis以守护进程模式运行
         pidfile              /var/run/redis_端口号.pid         设置redis的PID的文件位置
         port                  端口号                                    设置redis监听的端口号
         dir                    /var/redis/端口号                设置持久化文件存放位置
    d、现在可以使用/etc/init.d/redis_端口号start来启动redis了。
    e、执行下面命令使得redis随系统自动启动：
    $ sudo update-rc.d redis_端口号 defaults    //配置随机启动命令  redis_6380为初始化脚本文件

### 3、正确停止redis服务命令：
$ redis-cli SHUTDOWN /默认的  或者
$ redis-cli -p 6380 SHUTDOWN
当redis收到SHUTDOWN命令后，先断开所有客户端连接，然后根据配置执行持久化，最后完成退出。

### 4、关闭后再启动：

service redis_6380 -p 6380 start  #启动6380端口实例的redis

$ cd /etc/init.d && ./redis_6380 start  #默认启动6379端口实例的redis