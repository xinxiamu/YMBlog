---
title: 文件存储系统glusterfs入门
date: 2018-12-26 15:33:44
categories: 文件存储系统
tags: 
---

参考地址：https://www.gluster.org    
https://blog.csdn.net/phn_csdn/article/details/75153913       
https://yq.aliyun.com/articles/618056       
https://yq.aliyun.com/articles/553536?spm=a2c4e.11153940.blogcont618056.26.19cf227bo9Tp8P

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

## 其它命令

### 查看所volume（卷）

    gluster volume list
    
### 删除volume（卷）

    gluster volume stop models //停止名字为 models 的磁盘 
    gluster volume delete models //删除名字为 models 的磁盘                     