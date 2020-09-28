---
title: centos常用命令收藏
date: 2018-06-11 17:37:52
categories: CentOs
tags:
---

本文主要记录常用的命令……  

以备查询使用……

## 查看各种硬件信息

https://blog.csdn.net/dream_broken/article/details/52883883

## 日志

    /var/log/message 系统启动后的信息和错误日志，是Red Hat Linux中最常用的日志之一 
    /var/log/secure 与安全相关的日志信息 
    /var/log/maillog 与邮件相关的日志信息 
    /var/log/cron 与定时任务相关的日志信息 
    /var/log/spooler 与UUCP和news设备相关的日志信息
    /var/log/boot.log 守护进程启动和停止相关的日志消息
    
## 磁盘和分区

    # mount | column -t  # 查看挂接的分区状态 
    # fdisk -l   # 查看所有分区 
    # swapon -s   # 查看所有交换分区 
    # hdparm -i /dev/hda  # 查看磁盘参数(仅适用于IDE设备) 
    # dmesg | grep IDE  # 查看启动时IDE设备检测状况 
    
## 网络

    # ifconfig   # 查看所有网络接口的属性 
    # iptables -L   # 查看防火墙设置 
    # route -n   # 查看路由表 
    # netstat -lntp   # 查看所有监听端口 
    # netstat -antp   # 查看所有已经建立的连接 
    # netstat -s   # 查看网络统计信息 
    
## 用户

    # w    # 查看活动用户 
    # id <用户名>   # 查看指定用户信息 
    # last    # 查看用户登录日志 
    # cut -d: -f1 /etc/passwd # 查看系统所有用户 
    # cut -d: -f1 /etc/group # 查看系统所有组
    # crontab -l   # 查看当前用户的计划任务 
    
## 服务

    # chkconfig –list  # 列出所有系统服务 
    # chkconfig –list | grep on # 列出所有启动的系统服务                 

## 查看系统信息

    # uname -a   # 查看内核/操作系统/CPU信息 
    # cat /etc/issue 
    # cat /etc/redhat-release # 查看操作系统版本 
    # cat /proc/cpuinfo  # 查看CPU信息 
    # hostname   # 查看计算机名 
    # lspci -tv   # 列出所有PCI设备 
    # lsusb -tv   # 列出所有USB设备 
    # lsmod    # 列出加载的内核模块
    # env    # 查看环境变量 

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
    
## 查看服务器情况

### 1.查看服务器CPU型号  

    grep "model name" /proc/cpuinfo | cut -f2 -d:  
    
### 2.查看服务器内存容量

    grep MemTotal /proc/meminfo
    grep MemTotal /proc/meminfo | cut -f2 -d:
    free -m |grep "Mem" | awk '{print $2}' 
    
### 3.查看服务器的CPU是32位还是64位

    getconf LONG_BIT
    
### 4.查看当前Linux的版本

    more /etc/redhat-release cat /etc/redhat-release
    
### 5.查看Linux内核版本

    uname -r
    uname -a
    
### 6.查看服务器当前时间

    date
    
### 7.查看服务器硬盘和分区

    df -h
    fdisk -l
    
### 8.查看挂载情况

    mount
    
### 9.查看目录大小

    du /etc -sh  
    
### 10.查看服务器初始安装的软件包

    cat -n /root/install.log
    more /root/install.log | wc -l
    
### 11.查看已经安装的软件包

    rpm -qa
    rpm -qa | wc -l
    yum list installed | wc -l
    
### 12.查看服务器键盘布局

    cat /etc/sysconfig/keyboard
    cat /etc/sysconfig/keyboard | grep KEYTABLE | cut -f2 -d=
    
### 13.查看Selinux状态

    sestatus
    sestatus | cut -f2 -d:
    cat /etc/sysconfig/selinux
    
### 14.查看服务器网卡的ip，Mac地址,在ifcfg-eth0 文件里你可以看到mac，网关等信息。
    
    ifconfig
    cat /etc/sysconfig/network-scripts/ifcfg-eth0 | grep IPADDR
    cat /etc/sysconfig/network-scripts/ifcfg-eth0 | grep IPADDR | cut -f2 -d=
    ifconfig eth0 |grep "inet addr:" |awk '{print $2}'|cut -c 6-
    ifconfig | grep 'inet addr:'| grep -v '127.0.0.1' | cut -d: -f2 | awk '{ print $1}'
    
### 15.查看服务器默认网关

    cat /etc/sysconfig/network
    
### 16.查看服务器的默认DNS

    cat /etc/resolv.conf
    
### 17.查看服务器默认语言

    echo $LANG $LANGUAGE
    cat /etc/sysconfig/i18n
    
### 18.查看服务器所属时区和UTC时间

    cat /etc/sysconfig/clock
    
### 19.查看服务器主机名

    hostname
    cat /etc/sysconfig/network
    
### 20.查看文件大小  

    ll -h a.txt    
    
## CentOS挂载新硬盘                                        
参考：http://blog.sina.com.cn/s/blog_6177e8400101ntvu.html 
1.查看当前硬盘使用状况：

    df -h
2.查看新硬盘
 
    fdisk -l 
新添加的硬盘的编号为 `/dev/xvdb    /dev/xvde  /dev/vdb`

3.硬盘分区   
 1)进入fdisk模式     
    `/sbin/fdisk /dev/vdb`  
 2)输入n进行分区  
 3)选择分区类型   
 
  这里有两个选项：   
- p: 主分区 linux上主分区最多能有4个    
- e: 扩展分区 linux上扩展分区只能有1个，扩展分区创建后不能直接使用，还要在扩展分区上创建逻辑分区。     

这里我选择的p。

 4)选择分区个数  
 可以选择4个分区，这里我只分成1个分区    
 5)设置柱面，这里选择默认值就可以  
 6)输入w，写入分区表，进行分区   
 
4.格式化分区 

将新分区格式化为ext4文件系统     
1)如果创建的是主分区 
`mkfs -t ext4  /dev/vdb1 `

5.挂载硬盘  
1)创建挂载点     
在根目录下创建sqjr目录   
`mkdir /server /sqjr `
2)将/dev/vdb1挂载到/sqjr下    
 `mount /dev/vdb1 /server  /sqjr`
 
6.设置开机启动自动挂载    
新创建的分区不能开机自动挂载，每次重启机器都要手动挂载。    
设置开机自动挂载需要修改/etc/fstab文件    
`vi /etc/fstab `    
在文件的最后增加一行  
`/dev/vdb1 /server ext4 defaults 1 2 `

7.取消挂载 /dev/xvdb1   
`umount /dev/vdb1`

8.重启    
`reboot -n `                        

           
## 清理缓存，释放内存

1.清理yum缓存使用yum clean 命令，yum clean 的参数有headers, packages, metadata, dbcache, plugins, expire-cache, rpmdb, all

    yum clean headers  #清理/var/cache/yum的headers
    yum clean packages #清理/var/cache/yum下的软件包
    yum clean metadata
    ...
    
        
2.Linux释放内存

释放网页缓存(To free pagecache):    

    sync; echo 1 > /proc/sys/vm/drop_caches
    
释放目录项和索引(To free dentries and inodes):

    sync; echo 2 > /proc/sys/vm/drop_caches
    
释放网页缓存，目录项和索引（To free pagecache, dentries and inodes）:

    sync; echo 3 > /proc/sys/vm/drop_caches 
    
## 利用curl获取本机的外网ip

    #oray国内地址，返回速度快
    curl -s http://ddns.oray.com/checkip | awk -F ": " '{print $2}' | awk -F "\<" '{print $1}'
    #返回快
    curl ident.me
    curl myip.dnsomatic.com
    #返回较快
    curl whatismyip.akamai.com
    curl https://tnx.nl/ip
    #返回慢，不推荐
    curl ifconfig.me
    curl icanhazip.com
    curl ipecho.net/plain  
    
## 挂载磁盘

1.查看已挂载
```shell script
[root@xc-product-server-hn003 ~]# df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        3.7G     0  3.7G   0% /dev
tmpfs           3.7G     0  3.7G   0% /dev/shm
tmpfs           3.7G  564K  3.7G   1% /run
tmpfs           3.7G     0  3.7G   0% /sys/fs/cgroup
/dev/vda1        40G  4.3G   36G  11% /
overlay          40G  4.3G   36G  11% /var/lib/docker/overlay2/fe37037ef94f25079e4fdaeb9bc4485574cb309d032839b9638db0c1ffd88802/merged
tmpfs           755M     0  755M   0% /run/user/0
```

2.查看磁盘情况
```shell script
[root@xc-product-server-hn003 ~]# fdisk -l
Disk /dev/vda: 40 GiB, 42949672960 bytes, 83886080 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x4500abcf

Device     Boot Start      End  Sectors Size Id Type
/dev/vda1  *     2048 83886046 83883999  40G 83 Linux


Disk /dev/vdb: 100 GiB, 107374182400 bytes, 209715200 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```    

有磁盘`/dev/vdb`未挂载。下面挂载到目录`/server`下。

3.创建挂载点`/server`
```shell script
mkdir /server
```

4.格式化磁盘
```shell script
[root@xc-product-server-hn003 ~]# mkfs.ext4  /dev/vdb
mke2fs 1.45.4 (23-Sep-2019)
Creating filesystem with 26214400 4k blocks and 6553600 inodes
Filesystem UUID: 0a280208-edad-4186-8ded-ab5550fdb0dc
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
	4096000, 7962624, 11239424, 20480000, 23887872

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (131072 blocks): done
Writing superblocks and filesystem accounting information: done   
```

5.永久挂载
```shell script
vim /etc/fstab
```

添加一行：
`/dev/vdb 				  /server		  ext4	  defaults        0 0`

```text
# 
# /etc/fstab
# Created by anaconda on Mon Aug 24 06:23:59 2020
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
UUID=32f2af94-a1cd-4880-bb61-9ede22264d88 /                       xfs     defaults        0 0
/dev/vdb                                  /server                 ext4    defaults        0 0
```
保存，退出vim。

6.重启系统生效
```shell script
reboot
```

7.再次查看，看是否挂载成功
```shell script
[root@xc-product-server-hn003 ~]# clear
[root@xc-product-server-hn003 ~]# df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        3.7G     0  3.7G   0% /dev
tmpfs           3.7G     0  3.7G   0% /dev/shm
tmpfs           3.7G  472K  3.7G   1% /run
tmpfs           3.7G     0  3.7G   0% /sys/fs/cgroup
/dev/vda1        40G  4.3G   36G  11% /
/dev/vdb         98G   61M   93G   1% /server
tmpfs           755M     0  755M   0% /run/user/0
```
看到`/dev/vdb`已经挂载成功。
