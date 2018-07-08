---
title: 虚拟开发环境-Vagrant
date: 2018-01-23 10:04:27
categories: vagrant
tags: vagrant-start
---

介绍vagrant，为公司搭建统一的开发环境。最终会输出一个统一的虚拟开发环境，然后就可以分发给团队中所有的开发人员，大家在一致的开发环境中编辑，验证代码……

从此,告别“在我的机子上运行没问题的……”这个看似很有道理的扯皮了。

官网网址：https://www.vagrantup.com/

## vagrant安装

### 安装开源虚拟机VirtualBox

注意安装版本和vagrant版本要对应，否则后面会出现各种问题。这里安装最新版本。  
 
下载地址：https://www.virtualbox.org/wiki/Downloads 

选择自己电脑系统对应版本。

{% asset_img a.png %}

    axel -n 10 https://download.virtualbox.org/virtualbox/5.2.14/virtualbox-5.2_5.2.14-123301~Ubuntu~xenial_amd64.deb

然后，执行安装命令`sudo dpkg -i *.deb`进行安装即可。

安装过程中可能会缺少依赖包，把依赖包安装即可：

{% asset_img b.png %}

安装依赖包：`mutian@mutian-ThinkPad-T440p:~/Downloads$ sudo apt install libsdl1.2debian`

重新执行安装命令：

{% asset_img c.png %}

安装没有任何错误。 已安装到系统。

{% asset_img d.png %}

### 安装vagrant

安装最新版本。

下载地址：https://www.vagrantup.com/downloads.html 

下载自己电脑系统对应版本。

    axel -n 10 https://releases.hashicorp.com/vagrant/2.1.2/vagrant_2.1.2_x86_64.deb

然后，执行安装命令`sudo dpkg -i vagrant_2.1.2_x86_64.deb`进行安装即可。

查看安装是否成功：

    mutian@mutian-ThinkPad-T440p:~/Downloads$ vagrant -v
    Vagrant 2.1.2

表明安装已经成功，版本`2.1.2`。

## 添加Box

官方box源：https://app.vagrantup.com/boxes/search 

网络不好，需要翻墙。或者通过其它途径下载想要的box。

- 查看所有已下载box

`vagrant box list`

- 添加box

`vagrant box add centos/7`

执行上面命令会添加相应box到vagrant系统，如果没有，会先从box官源下载。 


## 创造自己box




## 学习资源

- http://www.imooc.com/learn/805