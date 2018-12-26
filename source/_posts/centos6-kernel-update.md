---
title: centos6升级内核
date: 2018-12-18 15:26:21
categories: CentOs
tags:
---

本章介绍linux系统升级内核的方法，在centos6环境下实验通过，并安装docker成功。     
如果不升级内核，安装docker，性能会收到影响。   

1.查看当前内核版本： 

    [root@iZwz9b0bqrkbhqd5lu2pwhZ ~]# uname -a
    Linux iZwz9b0bqrkbhqd5lu2pwhZ 2.6.32-696.10.1.el6.x86_64 #1 SMP Tue Aug 22 18:51:35 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
    [root@iZwz9b0bqrkbhqd5lu2pwhZ ~]# cat /proc/version
    Linux version 2.6.32-696.10.1.el6.x86_64 (mockbuild@c1bl.rdu2.centos.org) (gcc version 4.4.7 20120313 (Red Hat 4.4.7-18) (GCC) ) #1 SMP Tue Aug 22 18:51:35 UTC 2017
    
    -----------------------------------------------------------------
    [root@iZwz9b0bqrkbhqd5lu2pwhZ ~]# lsb_release -a
    LSB Version:	:base-4.0-amd64:base-4.0-noarch:core-4.0-amd64:core-4.0-noarch
    Distributor ID:	CentOS
    Description:	CentOS release 6.9 (Final)
    Release:	6.9
    Codename:	Final
    [root@iZwz9b0bqrkbhqd5lu2pwhZ ~]# cat /proc/version
    Linux version 2.6.32-696.10.1.el6.x86_64 (mockbuild@c1bl.rdu2.centos.org) (gcc version 4.4.7 20120313 (Red Hat 4.4.7-18) (GCC
    
2.导入public key    

     rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
     
3.安装ELRepo到CentOS 

可以去http://elrepo.org/tiki/tiki-index.php 选择要安装的ELRepo 

一般会安装到最新版内核

    rpm -Uvh https://www.elrepo.org/elrepo-release-6-8.el6.elrepo.noarch.rpm
    
4.安装 kernel-lt（lt=long-term）

    [root@localhost ~]# yum --enablerepo=elrepo-kernel install kernel-lt -y
    
或者 安装kernel-ml（ml=mainline）
 
    [root@localhost ~]# yum --enablerepo=elrepo-kernel install kernel-ml -y    
 
 
5.编辑grub.conf文件，修改Grub引导顺序     

    [root@localhost ~]# vim /etc/grub.conf
    
因为一般新安装的内核在第一个位置，所以设置default=0，表示启动新内核    


6.重启 

查看最新内核：

     [root@iZwz9b0bqrkbhqd5lu2pwhZ ~]# uname -a
     Linux iZwz9b0bqrkbhqd5lu2pwhZ 4.4.167-1.el6.elrepo.x86_64 #1 SMP Thu Dec 13 11:35:54 EST 2018 x86_64 x86_64 x86_64 GNU/Linux
     [root@iZwz9b0bqrkbhqd5lu2pwhZ ~]# uname -r
     4.4.167-1.el6.elrepo.x86_64
     
成功！      