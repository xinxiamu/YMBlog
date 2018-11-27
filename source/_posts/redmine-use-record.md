---
title: redmine使用踩坑记
date: 2018-11-23 09:14:54
categories: redmine
tags:
---

### 附件图片加载不全，中文附件下载空白

问题描述：   
在浏览器，域名访问redmine系统，图片显示不全，中文附件下载后空白。琢磨老半天，上服务器查看了下，附件都已经上传到服务器，从服务器把对应的图片拉取下来后，用工具打开，正常。我去，那是啥子问题咧，开始以为是Redmine系统编码问题，各种百度，么用。后来在本地环境装个Redmine环境，直接访问，不经过Nginx，不存在显示不了图片的问题，尴尬咯。那就只有一种可能了，Nginx配置的问题了，似乎看到了光明……

折腾了老半天，真他妈的是nginx的配置造成的。我去……     

原来配置：

    upstream redmine {
            server 127.0.0.1:8889;
            #server 127.0.0.1:8001;
            #server 127.0.0.1:8002;
    }
    
    server {
    	listen 80;
        server_name pm.xcsqjr.com;
        root /server/java/redmine/redmine-3.4.6/public;
    
        location / {
                try_files $uri @redmine;
        }

        location @redmine {
                proxy_set_header  X-Forwarded-For $remote_addr;
                proxy_pass http://redmine;
        }
    }
    
改为：

在conf中新增文件夹`sites`并添加文件`proxy.include`

    proxy_set_header   Host $http_host;                                                                                                                     
    proxy_set_header   X-Real-IP $remote_addr;                                                                                                                   
    proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header   X-Forwarded-Proto $scheme;
    
    client_max_body_size       10m;
    client_body_buffer_size    128k;
    
    proxy_connect_timeout      90;
    proxy_send_timeout         90;
    proxy_read_timeout         90;
    
    proxy_buffer_size          4k;
    proxy_buffers              4 32k;
    proxy_busy_buffers_size    64k;
    proxy_temp_file_write_size 64k;


    upstream redmine {
            server 127.0.0.1:8889;
            #server 127.0.0.1:8001;
            #server 127.0.0.1:8002;
    }

编辑配置：
   
    server {
    	listen 80;
        server_name pm.xcsqjr.com;
    	charset utf-8;
    	include sites/proxy.include;
         root /server/java/redmine/redmine-3.4.6/public;
    	#proxy_redirect off; #加上这行跳转会有问题
    
        location / {
                try_files $uri @redmine;
        }

        location @redmine {
                proxy_set_header  X-Forwarded-For $remote_addr;
                proxy_pass http://redmine;
        }
    } 
          