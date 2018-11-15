---
title: 通过Xshell登录vagrant虚拟机
date: 2018-11-15 15:00:40
categories: vagrant
tags:
---

在windows系统环境中，可以在命令行窗口中通过`vagrant ssh`登录`vagrant`虚拟机，但是操作有诸多不便，比如复制粘贴…… 
于是，我们想到了`Xshell`。下面介绍通过`Xshell`来登录`vagrant`虚拟机。 

## 使用`vagrant`账号登录

Vagrant虚拟机默认登录账号为`vagrant`,且通过私钥登录。 

在虚拟机 vagrantfile 的目录位置下进行。

1.启动虚拟机

    Xshell 6 (Build 0101)
    Copyright (c) 2002 NetSarang Computer, Inc. All rights reserved.
    
    Type `help' to learn how to use Xshell prompt.
    [d:\~]$ cd G:\xr-server\xr-server
    [G:\xr-server\xr-server]$ vagrant up
    Bringing machine 'default' up with 'virtualbox' provider...
    ==> default: Checking if box 'centos/7' is up to date...
    ==> default: Machine already provisioned. Run `vagrant provision` or use the `--provision`
    ==> default: flag to force provisioning. Provisioners marked to run always will still run.
    
    [G:\xr-server\xr-server]$ vagrant up
    Bringing machine 'default' up with 'virtualbox' provider...
    ==> default: Importing base box 'centos/7'...
    ==> default: Matching MAC address for NAT networking...
    ==> default: Checking if box 'centos/7' is up to date...
    ==> default: Setting the name of the VM: centos7_xr-server
    ==> default: Clearing any previously set network interfaces...
    ==> default: Preparing network interfaces based on configuration...
        default: Adapter 1: nat
    ==> default: Forwarding ports...
        default: 22 (guest) => 2222 (host) (adapter 1)
    ==> default: Running 'pre-boot' VM customizations...
    ==> default: Booting VM...
    ==> default: Waiting for machine to boot. This may take a few minutes...
        default: SSH address: 127.0.0.1:2222
        default: SSH username: vagrant
        default: SSH auth method: private key
        default: 
        default: Vagrant insecure key detected. Vagrant will automatically replace
        default: this with a newly generated keypair for better security.
        default: 
        default: Inserting generated public key within guest...
        default: Removing insecure key from the guest if it's present...
        default: Key inserted! Disconnecting and reconnecting using new SSH key...
    ==> default: Machine booted and ready!
    ==> default: Checking for guest additions in VM...
        default: No guest additions were detected on the base box for this VM! Guest
        default: additions are required for forwarded ports, shared folders, host only
        default: networking, and more. If SSH fails on this machine, please install
        default: the guest additions and repackage the box to continue.
        default: 
        default: This is not an error message; everything may continue to work properly,
        default: in which case you may ignore this message.
    ==> default: Setting hostname...
    ==> default: Rsyncing folder: /cygdrive/g/xr-server/xr-server/ => /vagrant

2.查看虚拟机ssh信息

    [G:\xr-server\xr-server]$ vagrant ssh-config
    Host default
      HostName 127.0.0.1
      User vagrant
      Port 2222
      UserKnownHostsFile /dev/null
      StrictHostKeyChecking no
      PasswordAuthentication no
      IdentityFile G:/xr-server/xr-server/.vagrant/machines/default/virtualbox/private_key
      IdentitiesOnly yes
      LogLevel FATAL

查看 hostname ，port，IdentityFile 这三个位置。知道登录主机，端口，登录私钥。

3.在Xshell下新建会话，登录

{%asset_img a-1.png%}

点击连接，如下图：

{%asset_img a-2.png%}

点击确定，如下图：

{%asset_img a-3.png%}

选定私钥。位置在`IdentityFile G:/xr-server/xr-server/.vagrant/machines/default/virtualbox/private_key`

登录成功：

    Connecting to 127.0.0.1:2222...
    Connection established.
    To escape to local shell, press 'Ctrl+Alt+]'.
    
    WARNING! The remote SSH server rejected X11 forwarding request.
    Last login: Thu Nov 15 07:31:28 2018 from 10.0.2.2
    [vagrant@xr-server ~]$ ll
    total 0
    [vagrant@xr-server ~]$ cd /
    [vagrant@xr-server /]$ 


## root账号登录

1.vagrant登陆后，切换到root账号，vagrant虚拟机的root账号密码默认为`vagrant`  
如果root没有初始化，则可以设置root的密码：   

    [vagrant@xr-server ~]$ su root
    Password: 
    [root@xr-server vagrant]# 

2.修改 /etc/ssh/sshd_config 文件，（注意，vagrant用户下这个文件是只读的，可能什么也看不见）   

- 修改 ssd_config 里 PermitRootLogin属性 改为yes ，并把前面的# 去掉

`[root@xr-server vagrant]# vim /etc/ssh/sshd_config`

{%asset_img b-1.png%}

保存退出。

- PasswordAuthentication 改为yes 并且去掉 #

{%asset_img b-2.png%}

保存退出。

3.保存退出，重启sshd服务     
`$ systemctl restart sshd`
或者
`systemctl restart sshd.service`

_问题_：虽然xshell里都是用127.0.0.1:2222或者2200 这种登录的，但是也可以使用自己设置的ip 例如192.16.25.11:22 去登录，这里用自己设置的ip时端口则是22。
设置完成以后就和自己开的虚拟机没什么两样了。


