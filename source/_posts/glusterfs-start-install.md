---
title: 文件存储系统glusterfs入门
date: 2018-12-26 15:33:44
categories: 文件存储系统
tags: 
---
    
参考地址：https://www.gluster.org 

https://my.oschina.net/hncscwc/blog/210072

## 原理


## 准备工作

用三个节点搭建一个glusterfs集群，一个节点作为客户端使用。新建四个虚拟机(centos7)：   
并编辑`etc/hosts`

     glusterfs服务器节点：  
     192.168.200.10  server-node1   # 主节点
     192.168.200.11  server-node2   # 从节点
     192.168.200.12  server-node3   # 从节点 
    
     client节点：   
     172.29.41.163 phn centos7

然后在每个机子互相ping，确保网络通畅：

在server-node1服务器执行： 

       # ping server-node2
       # ping server-node3 
       
其它机子同理，都ping下，保证互相通讯。


### 开始安装GlusterFS

####　安装服务

1.在每个服务节点安装：

    $ yum install -y centos-release-gluster
    $ yum install -y glusterfs glusterfs-server 
    $ yum install -y glusterfs-fuse glusterfs-rdma
    
2.设置服务开机自启动：  

    $ systemctl start glusterd.service && systemctl enable glusterd.service
    
    
#### 配置集群

1.首先启动服务（每个节点都启动）

    systemctl start glusterd.service 
    systemctl enable glusterd.service
    
2.关闭每个节点服务器的防火墙 

    systemctl stop firewalld.service 
    systemctl disable firewalld.service
    
3.在主节点中把另外两个从节点添加到集群    

    gluster peer probe server-node1 
    gluster peer probe server-node2
    
4.从集群中删除节点（在主节点服务器执行）

    $ gluster peer detach 192.168.200.12
    
可以从任意GFS Server节点上删除集群中的其它节点，但不能删除执行命令时的当前节点。

6.查看集群状态

在任意服务节点执行：  

    [root@server-node1 ~]# gluster peer status
    Number of Peers: 2
    
    Hostname: server-node2
    Uuid: 34bddd2b-4f4c-4b4b-afcd-786a6ba2c13d
    State: Peer in Cluster (Connected)
    
    Hostname: server-node3
    Uuid: 81101ab4-bcdc-467a-9fae-96d997a75fdc
    State: Peer in Cluster (Connected)

    ---------------------------------------------

    [root@server-node2 ~]# gluster peer status
    Number of Peers: 2
    
    Hostname: server-node1
    Uuid: d34a54a8-24bc-43ff-95fc-fe0b6d5c52fa
    State: Peer in Cluster (Connected)
    Other names:
    192.168.200.10
    
    Hostname: server-node3
    Uuid: 81101ab4-bcdc-467a-9fae-96d997a75fdc
    State: Peer in Cluster (Connected)
    
    ----------------------------------------------
    
    [root@server-node3 ~]# gluster peer status
    Number of Peers: 2
    
    Hostname: server-node1
    Uuid: d34a54a8-24bc-43ff-95fc-fe0b6d5c52fa
    State: Peer in Cluster (Connected)
    Other names:
    server-node1
    
    Hostname: server-node2
    Uuid: 34bddd2b-4f4c-4b4b-afcd-786a6ba2c13d
    State: Peer in Cluster (Connected)
    
在每个节点，可以看到另外的另个节点。

7.创建数据存储目录（每个节点服务都要创建）

    mkdir -p /data/gluster/exp1
    
8.创建卷，GFS Volume
  
在任意服务节点上执行如下命令：   

    gluster volume create models replica 3 server-node1:/data/gluster/exp1 server-node2:/data/gluster/exp1 server-node3:/data/gluster/exp1 force    
    
    ------------------------------------------
    [root@server-node1 gluster]# gluster volume create models replica 3 server-node1:/data/gluster/exp1 server-node2:/data/gluster/exp1 server-node3:/data/gluster/exp1 force
    volume create: models: success: please start the volume to access data
    
> _说明_:   
models: 卷的名称    
replica 3: 表明存储3个备份，后面指定服务器的存储目录    
    
- 查看volume 状态    
在任意服务节点执行：  


    gluster volume info     
    
    ------------------------------------------------------------
    
    [root@server-node2 ~]# gluster volume info
     
    Volume Name: models
    Type: Replicate
    Volume ID: 45d12287-2dfa-42be-954b-97c014d189c4
    Status: Created
    Snapshot Count: 0
    Number of Bricks: 1 x 3 = 3
    Transport-type: tcp
    Bricks:
    Brick1: server-node1:/data/gluster/exp1
    Brick2: server-node2:/data/gluster/exp1
    Brick3: server-node3:/data/gluster/exp1
    Options Reconfigured:
    transport.address-family: inet
    nfs.disable: on
    performance.client-io-threads: off
    
8.启动卷   
启动名为models的数据卷，在任意一个服务节点上执行即可：   
  
    gluster volume start models    
    
重新查看卷信息，会看到卷的状态status变为Started。代表卷已经启动成功。   

## 配置 GFS Client

GFS客户端的节点必须和各个服务节点网络连通。ping命令检查。

1.安装客户端

    $ yum install -y glusterfs glusterfs-fuse
    
2.将客户端目录挂载到GFS服务的volume 

- 在gluster客户节点创建本地目录：   

        $ mkdir -p /data/gluster/dt1
  
- 将本地目录挂载到GFS Volume：

        mount -t glusterfs 192.168.200.10:models /data/gluster/dt1
        
>参数说明：   
192.168.200.10： 指的是GFS服务主节点的ip，一定是主节点。
models： 指上面创建的GFS每个服务上的卷（volume）名称。   
/data/gluster/dt1: 客户端的本地目录，要挂载的目录。        

_注意_:   
上面挂载客户端目录到服务卷的命令可能不成功，这是因为用的是ip的原因，得把主节点的ip改成域名才行。因此，现在客户节点配置hosts。     
编辑`/etc/hosts`： 

    192.168.200.10  server-node1  #GFS Server主节点
    
然后重新挂载：

      mount -t glusterfs server-node1:models /data/gluster/dt1      

挂载成功。

3.查看挂载

    [root@gluster-client1 dt1]# df -h
    Filesystem                       Size  Used Avail Use% Mounted on
    /dev/mapper/VolGroup00-LogVol00   38G  3.3G   35G   9% /
    devtmpfs                         910M     0  910M   0% /dev
    tmpfs                            920M     0  920M   0% /dev/shm
    tmpfs                            920M  8.6M  911M   1% /run
    tmpfs                            920M     0  920M   0% /sys/fs/cgroup
    /dev/sda2                       1014M   63M  952M   7% /boot
    tmpfs                            184M     0  184M   0% /run/user/1000
    tmpfs                            184M     0  184M   0% /run/user/0
    server-node1:models               40G  3.4G   35G   9% /data/gluster/dt1
    
可以看到，最后一个文件系统，就是我们挂载的。
        
4.测试上传文件    
在客户机执行上传文件命令：   

    [root@gluster-client1 dt1]# time dd if=/dev/zero of=/data/gluster/dt1/hello bs=100M count=1
    1+0 records in
    1+0 records out
    104857600 bytes (105 MB) copied, 5.37804 s, 19.5 MB/s
    
    real	0m5.395s
    user	0m0.001s
    sys	0m0.182s
    [root@gluster-client1 dt1]# ls
    [root@gluster-client1 dt1]#    
 
查看GFS Server各个节点对应的卷，以及客户节点的本地目录：

    [root@server-node1 exp1]# pwd
    /data/gluster/exp1
    [root@server-node1 exp1]# ls
    hello
    
都能看到一个hello的文件。说明的确按我们预期，有三份的文件数据，分别在三个服务节点上存储了。同时在客户端节点也有一份。    
    
5.继续测试  
随便在客户节点本地目录下，创建文件`touch a`，然后查看各个服务节点，会发现，客户节点的a文件已经同步到三个服务的对应卷下了，编辑客户节点的a文件，各个服务节点对应的a文件也会相应改变。这说明了它们是一直同步的。      


## GFS性能调优

- 开启指定volume的配额： (models为volume名称)

 
    gluster volume quota models enable
    
- 限制models中 / (既总目录)最大使用80GB空间  

   
    gluster volume quota models limit-usage / 80GB   
    
- 设置 cache 大小(此处要根据实际情况，如果设置太大可能导致后面客户端挂载失败) 


    gluster volume set models performance.cache-size 512MB
    
- 开启异步，后台操作  

    
    gluster volume set models performance.flush-behind on
    
- 设置 io 线程 32 


    gluster volume set models performance.io-thread-count 32
    
- 设置 回写 (写数据时间，先写入缓存内，再写入硬盘)

    
    gluster volume set models performance.write-behind on
    
- 调优之后的volume信息


    gluster volume info
    
然后试下再上传文件，看是否快很多呢。                      

## 其它命令

### 查看所volume（卷）

    gluster volume list
    
### 删除volume（卷）

    gluster volume stop models //停止名字为 models 的磁盘 
    gluster volume delete models //删除名字为 models 的磁盘   


### 卸载GlusterFS磁盘

    -->gluster peer detach glusterfs4
    
### ACL访问控制

    -->gluster volume set models auth.allow 192.168.56.*,10.0.1.*

    volume set: success


### 添加GlusterFS节点
    -->gluster peer probe sc2-log5
    -->gluster peer probe sc2-log6
    -->gluster volume add-brick models sc2-log5:/data/gluster sc2-log6:/data/gluster

### 迁移GlusterFS数据

    -->gluster volume remove-brick models sc2-log1:/usr/local/share/models sc2-log5:/usr/local/share/models start
    -->gluster volume remove-brick models sc2-log1:/usr/local/share/models sc2-log5:/usr/local/share/models status
    -->gluster volume remove-brick models sc2-log1:/usr/local/share/models sc2-log5:/usr/local/share/models commit

### 修复GlusterFS数据(在节点1宕机的情况下)

    -->gluster volume replace-brick models sc2-log1:/usr/local/share/models sc2-log5:/usr/local/share/models commit -force
    -->gluster volume heal models full                      