---
title: shadowsocks-go代理服务器搭建
date: 2017-11-11 17:05:31
categories: shadowsocks
tags: shadowsocks-go
---

这是用来干嘛的，你懂的。

网址：

[shadowsocks](https://github.com/shadowsocks)

[sadowsocks-go](https://github.com/shadowsocks/shadowsocks-go)

## 服务端

编译好执行文件：{% asset_link server.tar.gz Shadowsocks-server %}  
放到服务器，更改.json文件配置，解压直接执行.sh文件即可。

1. 首先，买个国外的服务器再说吧……
   
2. 在服务器安装golang环境。
    安装包：{% asset_link go1.9.2.linux-amd64.tar.gz go1.9.2 %}
    
    这里不做介绍……
    
3. 在服务器安装git环境。

    这里不做介绍……
    
4. 下载服务端代码shadowsocks-go.

        # on server
        go get github.com/shadowsocks/shadowsocks-go/cmd/shadowsocks-server  
    
    golang环境会自动编译可执行代码到${GOPATH}/bin  
      
5.  执行启动

- *创建配置文件*：
`touch /server/shadowsocks/shadowsocks.json`
- *编辑shadowsocks.json*：
    
        {	 
            "server":"30.12.6.2",
            "server_port":8388,
            "local_port":1080,
            "password":"123456",
            "method": "aes-256-cfb",
            "timeout":600
        }

> 说明：       
`server`:服务器ip地址；
`server_port`:服务器端口；
`local_port:`客户端代理端口；
`method`:加密方式； 

- *启动*:


    shadowsocks-server -c /server/shadowsocks/shadowsocks.json > /server/shadowsocks/log &.   
    
> 说明：
`-c` 指定配置文件。 
`log`记录日志。
`&.` 后台执行    

- *查看是否启动*：

        [root@iZj6ca50pk1lwxqo14jss8Z ~]# netstat -lnp|grep 8388
        tcp6       0      0 :::8388                 :::*                    LISTEN      25719/shadowsocks-s 


---

## 客户端

### linux系统
编译好执行文件：{% asset_link client.tar.gz Shadowsocks-client %} 
在linux客户端解压，修改.json配置文件中相关参数，启动即可。

### windows系统
直接下载客户端。[网址](https://github.com/shadowsocks/shadowsocks-windows)
安装包：{% asset_link Shadowsocks-4.0.6.zip Shadowsocks-win %}
双击打开：
{% asset_img a.png %}

把服务器ip地址，还有设定的密码天上，确定即可。注意加密方法要和服务端设定的一致。

### ubuntu系统    

- go客户端：
`go get github.com/shadowsocks/shadowsocks-go/cmd/shadowsocks-local`
由于被墙，客户端下载不了。
所以要在服务端执行，然后再把可执行二进制文件下拉到本地。

*配置客户端文件*
创建文件：shadowsocks-local.json
编辑：
    
    {
    	"local_port": 1081,
    	"server_password": [
    		["127.0.0.1:8387", "foobar"],
    		["127.0.0.1:8388", "barfoo", "aes-128-cfb"]
    	]
    }

可以配置多个服务器，单个服务器，则去掉一个。

启动：类似服务端，用`-c`指定配置文件。    

- qt5客户端：
[安装网址](https://github.com/shadowsocks/shadowsocks-qt5/wiki/%E5%AE%89%E8%A3%85%E6%8C%87%E5%8D%97)  

### 移动

- 安卓客户端：

下载：{%asset_link Shadowsocks_v4.2.5_apkpure.com.apk Shadowsocks_v4.2.5_apkpure.com.apk%}

- 苹果客户端：

安装app：FirstWingy

### 修改浏览器代理
> SOCKS5 127.0.0.1:local_port

如果可以，在chrome中可以安装代理设置插件       
{% asset_img b.png %}

然后启动，试下访问：[google](https://www.google.com)




