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

- 再次执行`vagrant box list`便可以看到。 

## 创造自己box

- 创建目录

` mutian@mutian-ThinkPad-T440p:~/dev/vagrant$ mkdir xcsqjr-dev/` 

- 初始化，创建配置文件


    mutian@mutian-ThinkPad-T440p:~/dev/vagrant$ cd xcsqjr-dev/
    mutian@mutian-ThinkPad-T440p:~/dev/vagrant/xcsqjr-dev$ vagrant init
    A `Vagrantfile` has been placed in this directory. You are now
    ready to `vagrant up` your first virtual environment! Please read
    the comments in the Vagrantfile as well as documentation on
    `vagrantup.com` for more information on using Vagrant. 

会在当前目录下创建文件`Vagrantfile`。然后就可以编辑该文件，做一些配置：

    # -*- mode: ruby -*-
    # vi: set ft=ruby :
    
    # All Vagrant configuration is done below. The "2" in Vagrant.configure
    # configures the configuration version (we support older styles for
    # backwards compatibility). Please don't change it unless you know what
    # you're doing.
    Vagrant.configure("2") do |config|
      # The most common configuration options are documented and commented below.
      # For a complete reference, please see the online documentation at
      # https://docs.vagrantup.com.
    
      # Every Vagrant development environment requires a box. You can search for
      # boxes at https://vagrantcloud.com/search.
      
      # ----------- 一些相关配置 start ----------------#
      config.vm.box = "centos/7" #和已经下载的box名字一致
      config.vm.hostname = "ymu"  
      #config.vm.box_version = "1.1.0"	
      config.vm.box_url = "http://ymu.box"
    
      # 对虚拟机的一些配置 	
      config.vm.provider "virtualbox" do |vb|
      #   # Display the VirtualBox GUI when booting the machine
      #   vb.gui = true
      #
      #   # Customize the amount of memory on the VM:
          vb.memory = "1024" #为虚拟机分配内存
          vb.cpus = 2 #为虚拟机分配cup，分2核心
          vb.name = "centos7_ymu" #虚拟机名称
      end
      
      # ----------- 一些相关配置 end ----------------#
    
      # Disable automatic box update checking. If you disable this, then
      # boxes will only be checked for updates when the user runs
      # `vagrant box outdated`. This is not recommended.
      # config.vm.box_check_update = false
    
      # Create a forwarded port mapping which allows access to a specific port
      # within the machine from a port on the host machine. In the example below,
      # accessing "localhost:8080" will access port 80 on the guest machine.
      # NOTE: This will enable public access to the opened port
      # config.vm.network "forwarded_port", guest: 80, host: 8080
    
      # Create a forwarded port mapping which allows access to a specific port
      # within the machine from a port on the host machine and only allow access
      # via 127.0.0.1 to disable public access
      # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"
    
      # Create a private network, which allows host-only access to the machine
      # using a specific IP.
      # config.vm.network "private_network", ip: "192.168.33.10"
    
      # Create a public network, which generally matched to bridged network.
      # Bridged networks make the machine appear as another physical device on
      # your network.
      # config.vm.network "public_network"
    
      # Share an additional folder to the guest VM. The first argument is
      # the path on the host to the actual folder. The second argument is
      # the path on the guest to mount the folder. And the optional third
      # argument is a set of non-required options.
      # config.vm.synced_folder "../data", "/vagrant_data"
    
      # Provider-specific configuration so you can fine-tune various
      # backing providers for Vagrant. These expose provider-specific options.
      # Example for VirtualBox:
      #
      # config.vm.provider "virtualbox" do |vb|
      #   # Display the VirtualBox GUI when booting the machine
      #   vb.gui = true
      #
      #   # Customize the amount of memory on the VM:
      #   vb.memory = "1024"
      # end
      #
      # View the documentation for the provider you are using for more
      # information on available options.
    
      # Enable provisioning with a shell script. Additional provisioners such as
      # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
      # documentation for more information about their specific syntax and use.
      # config.vm.provision "shell", inline: <<-SHELL
      #   apt-get update
      #   apt-get install -y apache2
      # SHELL
    end

- 启动虚拟机

``    

## 学习资源

- http://www.imooc.com/learn/805
- https://github.com/apanly/mooc