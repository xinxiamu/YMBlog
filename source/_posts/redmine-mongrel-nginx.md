---
title: Redmine通过域名访问慢的问题解决
date: 2018-11-16 17:14:25
categories: redmine
tags: redmine-mongrel-nginx
---

按照之前章节在服务器安装好Redmine后，通过ngxin代理访问，发现巨慢，心塞！  
于是开始了各种搜，搜，搜……  
下面，把整个踩坑的过程记录下来！    

## Mongrel服务器启动Redmine
由于Redmine自带的Webrick Web服务器，听说是要进行域名代理解析，所以会特别慢。于是想确认下，通过翻墙访问，真他妹的变得快一些。唉，毕竟老外开发的东西呀，再加上咱国度的网络问题，呵呵了……   
还有，万能的网络告知还有个Mongrel的东东。    
Mongrel是一种快速的针对Ruby的Http服务器，专门为部署发布ROR应用而产生的。Mongrel相比Rails自带的纯Ruby服务器Webrick速度快很多并支持并发访问，有望成为Ruby的Tomcat.  

于是，开始各种google，百度，按照Mongrel

1.替换其自带的服务器webrick为mongrel，方法：  
`gem install mongrel`
rails 3.1以上执行:
`gem install mongrel --pre`

查看rails版本：
    
    [root@iZwz9b0bqrkbhqd5lu2pwhZ ~]# rails -v
    Rails 4.2.8

2.编辑`Gemfile.local`

进入Redmine安装目录下，新建文件`Gemfile.local`并编辑：

    [root@iZwz9b0bqrkbhqd5lu2pwhZ redmine-3.4.6]# cat Gemfile.local 
    # Gemfile.local 
    #gem 'thin'
    gem 'mongrel','~> 1.2.0.pre2'
    
3.删除gemfile.lock文件，重新执行:    
`bundle install`

4.重新启动Redmine：  
`ruby bin/rails server mongrel -e production -p 8889 -d`

## 配置Nginx代理

    upstream pm.xcsqjr.com{
        server localhost:8889;
        #server 10.162.71.10:5050;
        fair;
    }  
    server{
            listen 80;
            server_name pm.xcsqjr.com;
    	access_log  /server/java/nginx/logs/pm.xcsqjr.com.access.log;
            location ~ ${
                  server_name_in_redirect off;
    	      proxy_redirect off;
                  proxy_set_header Host $host;
                  proxy_set_header X-Real-IP $remote_addr;
                  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                  proxy_pass http://pm.xcsqjr.com;
                  client_max_body_size    100m;
                  index index.html index.htm;
            }
            error_page  404              /404.html;
            error_page  500 502 503 504  /50x.html;
            location = /50x.html {
                    root   html;
            }
    }
    
重启nginx，使配置生效。

激动人心的时刻即将到来，在浏览器输入：http://pm.xcsqjr.com/    
哎呀，他妹的，还是慢，慢，慢……    

于是，各种继续搜，搜，搜……，各种尝试，还是不行呀！！
 
哎呦，好像有点新发现了，在这里：    
https://www.nginx.com/resources/wiki/start/topics/recipes/redmine/?highlight=redmine    

好吧，就这样，把ngxin代理的配置改下，只能碰碰运气了：   

     upstream redmine {
             server 127.0.0.1:8889;
             #server 127.0.0.1:8001;
             #server 127.0.0.1:8002;
     }
     
     server {
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

没抱大希望咯，还是打开浏览器大神访问看看吧：http://pm.xcsqjr.com/     

啊，大爷的，飞速的快呀！那一刻，激动的泪水……

可以下班回家煮饭了，感谢上帝！！！