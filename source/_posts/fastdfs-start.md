---
title: fastdfs-分布式文件系统安装使用
date: 2018-02-01 10:51:36
categories: 文件存储系统
tags: 
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
    
停止:

    `/usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf stop`    
    
设置storage服务开机启动  

停止：

`/usr/bin/fdfs_storaged /etc/fdfs/storage.conf stop`       

……

## 测试
更改`/etc/fdfs/下client.conf配置文件。

`base_path=~/dev/fastdfs/data/client`   
`tracker_server=192.168.147` _不能是`localhost`或者`127.0.0.1`_

    mutian@mutian-ThinkPad-T440p:~$ /usr/bin/fdfs_test /etc/fdfs/client.conf upload /home/mutian/ifconfig.sh 
    This is FastDFS client test program v5.08
    
    Copyright (C) 2008, Happy Fish / YuQing
    
    FastDFS may be copied only under the terms of the GNU General
    Public License V3, which may be found in the FastDFS source kit.
    Please visit the FastDFS Home Page http://www.csource.org/ 
    for more detail.
    
    [2018-03-06 09:48:19] DEBUG - base_path=/home/mutian/dev/fastdfs/data/client, connect_timeout=30, network_timeout=60, tracker_server_count=1, anti_steal_token=0, anti_steal_secret_key length=0, use_connection_pool=0, g_connection_pool_max_idle_time=3600s, use_storage_id=0, storage server id count: 0
    
    tracker_query_storage_store_list_without_group: 
    	server 1. group_name=, ip_addr=192.168.1.146, port=23000
    
    group_name=group1, ip_addr=192.168.1.146, port=23000
    storage_upload_by_filename
    group_name=group1, remote_filename=M00/00/00/wKgBklqd82OAfszHAAAAHR6tZH87071.sh
    source ip address: 192.168.1.146
    file timestamp=2018-03-06 09:48:19
    file size=29
    file crc32=514679935
    example file url: http://192.168.1.146/group1/M00/00/00/wKgBklqd82OAfszHAAAAHR6tZH87071.sh
    storage_upload_slave_by_filename
    group_name=group1, remote_filename=M00/00/00/wKgBklqd82OAfszHAAAAHR6tZH87071_big.sh
    source ip address: 192.168.1.146
    file timestamp=2018-03-06 09:48:19
    file size=29
    file crc32=514679935
    example file url: http://192.168.1.146/group1/M00/00/00/wKgBklqd82OAfszHAAAAHR6tZH87071_big.sh
    mutian@mutian-ThinkPad-T440p:~$ 
    
    ####
    mutian@mutian-ThinkPad-T440p:~/Pictures$ /usr/bin/fdfs_upload_file /etc/fdfs/client.conf aa.png 
    group1/M00/00/00/wKgBklqgqI6AbB6yAAGUbXc7zK4788.png
    mutian@mutian-ThinkPad-T440p:~/Pictures$ cd
    mutian@mutian-ThinkPad-T440p:~$ /usr/bin/fdfs_upload_file /etc/fdfs/client.conf south_air.zip 
    group1/M00/00/00/wKgBklqgqSaABpepFCjSUGWmckg821.zip
    

## ubuntu下搭建

验证过，上面过程适用…… 

## 集成nginx模块

参考：https://github.com/happyfish100/fastdfs-nginx-module/blob/master/INSTALL

1.下载`fastdfs-nginx-module`

`git clone https://github.com/happyfish100/fastdfs-nginx-module.git`

注意：安装的FastDFS版本 >= 5.11

2.安装nginx-1.8.1

下载：http://nginx.org/en/download.html

    > ./configure --prefix=~/nginx \
     --add-module=/home/mutian/fastdfs-nginx-module/src
    
    > make; make install
    
3.更改nginx配置，添加一行。

如果文件分组  
        
    location ~/group([0-9])/M00 {
    
        ngx_fastdfs_module;
    
    }

如果没分组

    location /M00 {
        root /home/mutian/dev/fastdfs/data/storage/data;
        ngx_fastdfs_module;
    }
    
> 注意：
 A、8888 端口值是要与/etc/fdfs/storage.conf 中的 http.server_port=8888 相对应, 因为 http.server_port 默认为 8888,如果想改成 80,则要对应修改过来。
 B、Storage 对应有多个 group 的情况下,访问路径带 group 名,如/group1/M00/00/00/xxx, 对应的 Nginx 配置为:
 location ~/group([0-9])/M00 {
     ngx_fastdfs_module;
 }
    
    
4.拷贝fdfs_storage的文件存储软链接

`ln -s /home/mutian/dev/fastdfs/data/storage/data  /home/mutian/dev/fastdfs/data/storage/data/M00`          

5.更改配置`mod_fastdfs.conf`

拷贝到相关目录：

    cp ~/dev/fastdfs/fastdfs-nginx-module/src/mod_fastdfs.conf /etc/fdfs/  
    
更改内容：

    connect_timeout=10
    base_path=/tmp
    tracker_server=ip01:22122
    storage_server_port=23000
    group_name=group1
    url_have_group_name = true
    store_path0=/home/mutian/dev/fastdfs/data/storage      
    
6.复制FastDFS 的部分配置文件到`/etc/fdfs`目录

    mutian@mutian-ThinkPad-T440p:~$ cd /home/mutian/dev/fastdfs/fastdfs-5.11/conf/
    mutian@mutian-ThinkPad-T440p:~/dev/fastdfs/fastdfs-5.11/conf$ ls
    anti-steal.jpg  http.conf   storage.conf      tracker.conf
    client.conf     mime.types  storage_ids.conf

    cp http.conf mime.types /etc/fdfs/   
    
7.启动nginx

`~/dev/nginx/sbin/nginx -s stop; ~/dev/nginx/sbin/nginx` 

8.测试
按上面步骤，上传个文件，然后在浏览器打开：

http://ip:port/group1/M00/00/00/tlxkwlhttsGAU2ZXAAC07quU0oE095.png 

or 

http://ip/group1/M00/00/00/tlxkwlhttsGAU2ZXAAC07quU0oE095.png          