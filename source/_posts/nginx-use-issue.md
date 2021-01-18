---
title: nginx使用问题收藏
date: 2019-01-25 11:52:40
categories: nginx
tags:
---

## nginx出现 “414 request-uri too large”

客户端请求头缓冲区大小，如果请求头总长度大于小于128k，则使用此缓冲区，

请求头总长度大于128k时使用large_client_header_buffers设置的缓存区
client_header_buffer_size 128k;

large_client_header_buffers 指令参数4为个数，128k为大小，默认是8k。申请4个128k。
large_client_header_buffers 4 128k

解决办法：   
在nginx的nginx.conf修改如下参数的：

client_header_buffer_size 512k;
large_client_header_buffers 4 512k;

然后重新启动nginx
     
## 上传文件失败 Failed to load resource: the server responded with a status of 413 (Request Entity Too Large)    

报错问题：

Failed to load resource: the server responded with a status of 413 (Request Entity Too Large) 

解决：

 打开nginx.conf配置文件，修改client_max_body_size值   
 加上下面这行：

 client_max_body_size   30M（改成你想要的数值） 
 
 然后重新启动nginx
 
 _提醒：_  
 
 于是奇葩的问题被我们遇到了，详细配置请参考下面。我们的问题是，无论client_max_body_size设置在哪里，nginx －s reload后，依然一直报413.多次尝试reload，始终无效。最终决定kill 进程，restart，终于好了。
 
## nginx常用命令

#### 检查配置是否正确

```shell script
./nginx -t
```

#### 启动、停止、重启nginx

