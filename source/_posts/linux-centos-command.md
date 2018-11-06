---
title: centos常用命令收藏
date: 2018-06-11 17:37:52
categories: CentOs
tags: centos常用命令
---

本文主要记录常用的命令……  

以备查询使用……


## 查看系统信息

### 查看端口占用

`netstat -lnp|grep 8000`

## 设置自启动

在生产环境，一些基础应用环境需要系统启动的时候自动启动……

### 服务自启动设置

执行命令`systemctl enable *`即可。

例如设置`docker`服务自启动：  
`systemctl enable docker.service`

### 脚本自启动

### 参考：

    一、添加开机自启服务
    
    在CentOS 7中添加开机自启服务非常方便，只需要两条命令(以Jenkins为例)：
    systemctl enable jenkins.service #设置jenkins服务为自启动服务
    sysstemctl start  jenkins.service #启动jenkins服务
    
    二、添加开机自启脚本
    
    在centos7中增加脚本有两种常用的方法，以脚本autostart.sh为例：
    #!/bin/bash
    #description:开机自启脚本
    /usr/local/tomcat/bin/startup.sh  #启动tomcat
    
    方法一
    
    1、赋予脚本可执行权限（/opt/script/autostart.sh是你的脚本路径）
    chmod +x /opt/script/autostart.sh
    
    2、打开/etc/rc.d/rc/local文件，在末尾增加如下内容
    /opt/script/autostart.sh
    
    3、在centos7中，/etc/rc.d/rc.local的权限被降低了，所以需要执行如下命令赋予其可执行权限
    chmod +x /etc/rc.d/rc.local
    
    方法二
    
    1、将脚本移动到/etc/rc.d/init.d目录下
    mv  /opt/script/autostart.sh /etc/rc.d/init.d
    
    2、增加脚本的可执行权限
    chmod +x  /etc/rc.d/init.d/autostart.sh
    
    3、添加脚本到开机自动启动项目中
    cd /etc/rc.d/init.d
    chkconfig --add autostart.sh
    chkconfig autostart.sh on
    
## 系统进程管理

### 查看进程信息

#### 1.根据端口，查看进程信息

    # lsof -i:8011  //查看端口，找到进程pid
    # cd /proc/pid
    # ll
    
    ---------------------------------------------------------
    
    [root@izwz9bwlp4pyxy4mnxaycez 1113]# ll
    total 0
    dr-xr-xr-x  2 root root 0 10月 31 17:30 attr
    -rw-r--r--  1 root root 0 11月  5 10:24 autogroup
    -r--------  1 root root 0 11月  5 10:24 auxv
    -r--r--r--  1 root root 0 11月  5 10:24 cgroup
    --w-------  1 root root 0 11月  5 10:24 clear_refs
    -r--r--r--  1 root root 0 10月  8 21:02 cmdline
    -rw-r--r--  1 root root 0 11月  5 10:24 comm
    -rw-r--r--  1 root root 0 11月  5 10:24 coredump_filter
    -r--r--r--  1 root root 0 11月  5 10:24 cpuset
    lrwxrwxrwx  1 root root 0 11月  5 10:24 cwd -> /
    -r--------  1 root root 0 11月  1 15:13 environ
    lrwxrwxrwx  1 root root 0 11月  5 10:24 exe -> /server/java/jdk/bin/java
    dr-x------  2 root root 0 10月 31 17:30 fd
    dr-x------  2 root root 0 11月  1 11:53 fdinfo
    -rw-r--r--  1 root root 0 11月  5 10:24 gid_map
    -r--------  1 root root 0 11月  5 10:24 io
    -r--r--r--  1 root root 0 11月  5 10:24 limits
    -rw-r--r--  1 root root 0 11月  5 10:24 loginuid
    dr-x------  2 root root 0 11月  5 10:24 map_files
    -r--r--r--  1 root root 0 11月  5 10:24 maps
    -rw-------  1 root root 0 11月  5 10:24 mem
    -r--r--r--  1 root root 0 11月  5 10:24 mountinfo
    -r--r--r--  1 root root 0 11月  5 10:24 mounts
    -r--------  1 root root 0 11月  5 10:24 mountstats
    dr-xr-xr-x  5 root root 0 11月  5 10:24 net
    dr-x--x--x  2 root root 0 11月  5 10:24 ns
    -r--r--r--  1 root root 0 11月  5 10:24 numa_maps
    -rw-r--r--  1 root root 0 11月  5 10:24 oom_adj
    -r--r--r--  1 root root 0 11月  5 10:24 oom_score
    -rw-r--r--  1 root root 0 11月  5 10:24 oom_score_adj
    -r--r--r--  1 root root 0 11月  5 10:24 pagemap
    -r--------  1 root root 0 11月  5 10:24 patch_state
    -r--r--r--  1 root root 0 11月  5 10:24 personality
    -rw-r--r--  1 root root 0 11月  5 10:24 projid_map
    lrwxrwxrwx  1 root root 0 11月  5 10:24 root -> /
    -rw-r--r--  1 root root 0 11月  5 10:24 sched
    -r--r--r--  1 root root 0 11月  5 10:24 schedstat
    -r--r--r--  1 root root 0 11月  5 10:24 sessionid
    -rw-r--r--  1 root root 0 11月  5 10:24 setgroups
    -r--r--r--  1 root root 0 11月  5 10:24 smaps
    -r--r--r--  1 root root 0 11月  5 10:24 stack
    -r--r--r--  1 root root 0 10月  8 21:02 stat
    -r--r--r--  1 root root 0 11月  5 10:24 statm
    -r--r--r--  1 root root 0 10月  8 21:02 status
    -r--r--r--  1 root root 0 11月  5 10:24 syscall
    dr-xr-xr-x 30 root root 0 11月  5 10:24 task
    -r--r--r--  1 root root 0 11月  5 10:24 timers
    -rw-r--r--  1 root root 0 11月  5 10:24 uid_map
    -r--r--r--  1 root root 0 11月  5 10:24 wchan

#### 2.查看进程

    # ps aux | less  --查看所有运行中的进程
    # ps -A --查看系统中的每个进程。
    # ps -u vivek --查看用户vivek运行的进程
    
#### 3.动态显示进程

    # top
按q退出，按h进入帮助

#### 4.树状显示进程

    [root@izwz9bwlp4pyxy4mnxaycez ~]# pstree
    systemd─┬─aliyun-service
            ├─atd
            ├─auditd───{auditd}
            ├─crond
            ├─dbus-daemon
            ├─dhclient
            ├─irqbalance
            ├─26*[java───27*[{java}]]
            ├─java───26*[{java}]
            ├─2*[java───32*[{java}]]
            ├─java───23*[{java}]
            ├─java───21*[{java}]
            ├─java───61*[{java}]
            ├─java───247*[{java}]
            ├─java───57*[{java}]
            ├─java───49*[{java}]
            ├─login───bash
            ├─mysqld_safe───mysqld───128*[{mysqld}]
            ├─mysqld_safe───mysqld───194*[{mysqld}]
            ├─10*[nginx───nginx]
            ├─ntpd
            ├─polkitd───5*[{polkitd}]
            ├─redis-server───2*[{redis-server}]
            ├─rsyslogd───2*[{rsyslogd}]
            ├─sshd─┬─3*[sshd───bash]
            │      ├─2*[sshd───bash───2*[tail]]
            │      ├─sshd───bash───pstree
            │      └─sshd───sshd
            ├─systemd-journal
            ├─systemd-logind
            ├─systemd-udevd
            ├─tuned───4*[{tuned}]
            └─wrapper─┬─java───53*[{java}]
                      └─{wrapper}

### 将进程快照储存到文件中

输入下列命令：
    	
    # top -b -n1 > /tmp/process.log

你也可以将结果通过邮件发给自己：

    # top -b -n1 | mail -s 'Process snapshot' you@example.com
    
        