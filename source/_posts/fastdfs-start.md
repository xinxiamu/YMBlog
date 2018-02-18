---
title: fastdfs-分布式文件系统安装使用
date: 2018-02-01 10:51:36
categories: 文件存储系统
tags: fastdfs-安装使用
---

本文记录FastDFS分布式文件系统的在服务器的搭建，配置以及使用……

网址：https://github.com/happyfish100/fastdfs

参考：
http://blog.csdn.net/xyang81/article/details/52837974
http://blog.csdn.net/playadota/article/details/78381109

## centos下搭建

### 第一步：安装依赖
安装fastdfs之前，先要安装相关依赖包libfastcommon。 

下载地址：https://github.com/happyfish100/libfastcommon.git

`git clone https://github.com/happyfish100/libfastcommon.git`

编译安装：

    > cd libfastcommon
    > ./make.sh
    > ./make.sh install

### 第二部：安装fastdfs
下载：https://github.com/happyfish100/fastdfs

- step 2. download FastDFS source package and unpack it, 
tar xzf FastDFS_v5.x.tar.gz

- step 3. enter the FastDFS dir

    `cd FastDFS`

- step 4. execute:

    `./make.sh`

- step 5. make install

    ./make.sh install`

- step 6. edit/modify the config file of tracker and storage


    cd /etc/fdfs/
    cp tracker.conf.sample tracker.conf
    cp storage.conf.sample storage.conf
    mkdir -p /server/data/fdfs

首先修改配置文件：  /etc/fdfs/tracker.conf，修改路径到/server/data/fdfs目录。

base_path=/server/data/fdfs/tracker  

启动：
    
    #start the tracker server:
    /usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf restart
    #in Linux, you can start fdfs_trackerd as a service:
    /sbin/service fdfs_trackerd start 
    
检查启动是否成功：
    `ps -ef | grep fdfs_trackerd `    
    
设置tracker服务开启启动：
    

--------------
修改配置文件：  /etc/fdfs/storag.conf，修改路径到/server/data/fdfs目录，同时配置tracker_server地址。

        # the base path to store data and log files
        base_path=/server/data/fdfs/storeage
        # tracker_server can ocur more than once, and tracker_server format is
        #  "host:port", host can be hostname or ip address
        tracker_server=192.168.1.36:22122
        # store_path#, based 0, if store_path0 not exists, it's value is base_path
        # the paths must be exist
        store_path0=/server/data/fdfs/storeage
        #store_path1=/home/yuqing/fastdfs2
        
启动：
    
    #start the storage server:
    /usr/bin/fdfs_storaged /etc/fdfs/storage.conf restart
    #in Linux, you can start fdfs_storaged as a service:
    /sbin/service fdfs_storaged start  
    
检查启动是否成功：
    `ps -ef | grep fdfs_storaged `  
    
设置storage服务开机启动        

……

##     
    