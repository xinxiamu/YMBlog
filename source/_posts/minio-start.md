---
title: 分布式文件存储系统 MINIO入门
date: 2020-06-18 11:05:36
categories: 文件存储系统
tags:
---

发现一个比FastDFS更加简单、体验更加好的文件存储系统，撸它……

## 安装-Docker环境下

```shell script
docker run -p 9000:9000 -p 9001:9001 \
--restart=always \
--name minio1 \
-v /mnt/data:/data \
-e "MINIO_ROOT_USER=izajzo6yycy5d65vexjmdvtts" \
-e "MINIO_ROOT_PASSWORD=kvb9fv35c14soranzsz8bwzyf" \
-d minio/minio server /data --console-address ":9001"
```

安装完毕，可以浏览器打开管理界面：http://localhost:9001

_最新_:

```shell
docker run -p 9000:9000 -p 9001:9001 \
--restart=always \
--name minio1 \
-v /mnt/data:/data \
-e "MINIO_ROOT_USER=izajzo6yycy5d65vexjmdvtts" \
-e "MINIO_ROOT_PASSWORD=kvb9fv35c14soranzsz8bwzyf" \
-d quay.io/minio/minio server /data --console-address ":9001"
```

## 客户端使用

1.安装客户端

首先是安装minio客户端，这个官方文档很清楚，各取所需吧.(Linux 或者windows 选择下载一个即可)

```text
Linux 二进制文件地址：https://dl.minio.io/client/mc/release/linux-amd64/mc

windows exe文件：https://pan.baidu.com/s/1DxO0MgXqtEVg40FgiyL1CQ
```

2.设置自定义命令并启动

```text
Linux下： 
chmod +x ./mc
alias mc="./mc"

windows下： mc.exe
```

至此，我们的minio client就安装完成了。

3.添加服务端host

```text
使用 minio client 将我自己的 minio server 添加到 mc 的配置管理：

accessKey：admin   

secretKey:  password
```

```shell script
[root@xr-server-dev minio]# ls
mc
[root@xr-server-dev minio]# ./mc config host add minio1 http://192.168.0.3:9000 izajzo6yycy5d65vexjmdvtts kvb9fv35c14soranzsz8bwzyf
Added `minio1` successfully.
```

查看：

```shell script
[root@xr-server-dev minio]# ./mc config host list
gcs   
  URL       : https://storage.googleapis.com
  AccessKey : YOUR-ACCESS-KEY-HERE
  SecretKey : YOUR-SECRET-KEY-HERE
  API       : S3v2
  Path      : dns

local 
  URL       : http://localhost:9000
  AccessKey : 
  SecretKey : 
  API       : 
  Path      : auto

minio1
  URL       : http://192.168.0.3:9000
  AccessKey : izajzo6yycy5d65vexjmdvtts
  SecretKey : kvb9fv35c14soranzsz8bwzyf
  API       : s3v4
  Path      : auto

play  
  URL       : https://play.min.io
  AccessKey : Q3AM3UQ867SPQQA43P2F
  SecretKey : zuf+tfteSlswRu7BJ86wekitnifILbZam1KYY3TG
  API       : S3v4
  Path      : auto

s3    
  URL       : https://s3.amazonaws.com
  AccessKey : YOUR-ACCESS-KEY-HERE
  SecretKey : YOUR-SECRET-KEY-HERE
  API       : S3v4
  Path      : dns


```

可以看到minio的host。

#### 设置永久下载策略

配置策略命令查看: mc policy

```shell script
[root@xc-product-server-hn002 ~]# ./mc policy set download minio/xrlj-20210113
Access permission for `minio/xrlj-20210113` is set to `download`
```

这个命令的作用是将 server 端的 mybucket 桶设置为开放管理，可以直接通过 url 进行下载。

[桶名]/[路径]可以一直拼接到具体的文件夹或文件

类似于以下 http://xxx.xxx.xxx.xxx:9000/mybucket/xxx.zip，可用浏览器直接从此URL访问下载。

### 上传文件

### 查看文件

## 集成nginx访问文件

## 参考资源

[官网](https://min.io/)
