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

1. 二进制包安装。[安装网址](https://github.com/shadowsocks/shadowsocks-qt5/wiki/%E5%AE%89%E8%A3%85%E6%8C%87%E5%8D%97)  

2. 推荐，命令安装：
    
        #Ubuntu 14.04及更高版本
        #添加ppa源
        sudo add-apt-repository ppa:hzwhuang/ss-qt5
        sudo apt-get update
        sudo apt-get install shadowsocks-qt5
         
        #启动shadowsocks-qt5
        
        可以通过which shadowsocks-qt5找到可执行文件的位置。
        
        执行 ./shadowsocks-qt5(桌面板，可以通过搜索已安装的shadowsocks-qt5，点击图标启动) 

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



### linux下全局代理

实验环境：Ubuntu系统

1.安装应用`privoxy`

一款工具，将socks代理转换成http。

`sudo apt-get install privoxy`

2.更改配置

    sudo gedit /etc/privoxy/config
    
    ----------------------------------------------------------------

    # 在 froward-socks4下面添加一条socks5的，因为shadowsocks为socks5，
    # 地址是127.0.0.1:1080。注意他们最后有一个“.”
    #        forward-socks4   /               socks-gw.example.com:1080  .
    forward-socks5   /               127.0.0.1:1080 .
    
    # 下面还存在以下一条配置，表示privoxy监听本机8118端口，
    # 把它作为http代理，代理地址为 http://localhost.8118/ 。
    # 可以把地址改为 0.0.0.0:8118，表示外网也可以通过本机IP作http代理。
    # 这样，你的外网IP为1.2.3.4，别人就可以设置 http://1.2.3.4:8118/ 为http代理。
    　listen-address localhost:8118  #端口可以随意设定
    
    上面配置可能导致无法启动privoxy,新版本安全问题导致。改为：   
    listen-address 192.168.1.115:8118  #端口可以随意设定  

3.重启privoxy

    sudo systemctl restart privoxy.serivce
    
4.添加环境变量

    vim ~/.bashrc
    
    ----
    添加两行：
    export http_proxy=http://127.0.0.1:8118/
    export https_proxy=http://127.0.0.1:8118/  
    
5.使环境变量立即生效

    source ~/.bashrc
    
说明：如果只是想临时的让当前命令窗口代理，那么只需要添加临时变量，不需要编辑~/.bashrc。  
只需要在当前命令窗口执行`export http_proxy=http://127.0.0.1:8118/` ` export https_proxy=http://127.0.0.1:8118/ `         


## Shadowsocks-rust 搭建vpn

[网址](https://github.com/shadowsocks/shadowsocks-rust)

1. 服务器安装`rust`环境

```shell
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh 
source "$HOME/.cargo/env"
rustc -V
rustc 1.62.0 (a8314ef7d 2022-06-27)
```

会下载脚本文件并执行，安装rust。

2. 安装Shadowsocks-rust服务和客户端

执行命令：
```shell
cargo install shadowsocks-rust
```

报错：
```shell
error: linker `cc` not found
  |
  = note: No such file or directory (os error 2)

error: could not compile `libc` due to previous error
warning: build failed, waiting for other jobs to finish...

```

是因为linux服务器缺少c编译环境，可以执行下面命令安装：
```shell
sudo yum -y install gcc gcc-c++ kernel-devel
```

安装成功后，查看安装好的安装包：
```shell
[root@server-dev-01 .cargo]# pwd
/root/.cargo
[root@server-dev-01 .cargo]# ls
bin  env  registry
[root@server-dev-01 .cargo]# cd bin/
[root@server-dev-01 bin]# ls
cargo         cargo-fmt   clippy-driver  rustc    rustfmt   rust-gdbgui  rustup   ssmanager  ssservice
cargo-clippy  cargo-miri  rls            rustdoc  rust-gdb  rust-lldb    sslocal  ssserver   ssurl
[root@server-dev-01 bin]# 

```

两个服务。`ssserver`服务端。`sslocal`客户端。

3. 配置，试用：

新建配置文件：shadowsocks.json，编辑内容：
```text
{
    "servers": [
        {
            "address": "192.168.0.106",
            "port": 8388,
            "password": "a1234567",
            "method": "aes-256-gcm",
            "timeout": 7200
        },
        {
            "address": "192.168.0.106",
            "port": 8389,
            "password": "a1234567",
            "method": "chacha20-ietf-poly1305"
        },
        {
            "disabled": true,
            "address": "eg.disable.me",
            "port": 8390,
            "password": "hello-internet",
            "method": "chacha20-ietf-poly1305"
        }
    ],
    // ONLY FOR `sslocal`
    // Delete these lines if you are running `ssserver` or `ssmanager`
    "local_port": 1080,
    "local_address": "127.0.0.1"
}

```

可以配置多个服务实例。不同的端口运行不同的实例。同一个进程。客户端连的时候，会选择延迟最小，高可用的连接。

后台启动服务：
```shell
ssserver -c shadowsocks.json -d 
```

客户端连接vpn服务，可以用sslocal客户端，也可以用其它的vpn客户端，比如各个平台的vpn界面服务端。
