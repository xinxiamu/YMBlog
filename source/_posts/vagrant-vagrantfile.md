---
title: vagrant常用配置介绍
date: 2018-11-15 17:55:52
categories: vagrant
tags:
---

https://www.vagrantup.com/docs/vagrantfile/

## 基础配置

    config.vm.box = "centos/7" 
基础镜像，本机没有，会从网络拉取    

     config.vm.hostname = "xr-server" 
很重要，多来虚拟机，用来区别。可以通过vagrant up centos1来指定只启动哪一台。

## 虚拟机网络设置

    config.vm.network "private_network", ip: "192.168.10.11"
    //Host-only模式
     
    config.vm.network "public_network", ip: "10.1.2.61"
    //Bridge模式

Vagrant的网络连接方式有三种：  

- NAT : 缺省创建，用于让vm可以通过host转发访问局域网甚至互联网。
- host-only : 只有主机可以访问vm，其他机器无法访问它。
- bridge : 此模式下vm就像局域网中的一台独立的机器，可以被其他机器访问。


    config.vm.network :private_network, ip: "192.168.33.10"
    #配置当前vm的host-only网络的IP地址为192.168.33.10

host-only 模式的IP可以不指定，而是采用dhcp自动生成的方式，如 :    

    config.vm.network "private_network", type: "dhcp”

```text
#config.vm.network "public_network", ip: "192.168.0.17"
#创建一个bridge桥接网络，指定IP
#config.vm.network "public_network", bridge: "en1: Wi-Fi (AirPort)"
#创建一个bridge桥接网络，指定桥接适配器
config.vm.network "public_network"
#创建一个bridge桥接网络，不指定桥接适配器

```

## 同步目录设置

网址：https://www.vagrantup.com/docs/synced-folders/

默认是：宿主机子`vagrantfile`所在目录，虚拟机的`/vagrant`目录 。

     config.vm.synced_folder "D:/xxx/code", "/home/www/" 

第一个地址为宿主机目录，第二个为虚拟机的目录。

## 端口转发

