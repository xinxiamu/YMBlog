---
title: 项目管理系统Redmine使用
date: 2018-06-01 21:35:04
categories: redmine
tags: 羡慕管理系统
---

在软件开发过程中，项目管理的有效性直接会影响到项目开发的进度，质量，还有整个团队的协助。总之，没有一个好的项目管理系统，这对项目经理来说就像是上了战场，却没有任何武器一样，必定败仗。  

下面介绍这款比较流行的项目管理工具redmine。有了它，项目经理就可以对整个项目的开发全过程进行管理。任务的分配、bug的跟踪、知识库的建立等等……

[官网](http://www.redmine.org)

## 在ubuntu下安装

参考：http://www.redmine.org/projects/redmine/wiki/Guide

###　下载redmine安装包

网址：http://www.redmine.org/projects/redmine/wiki/Download

    wget http://www.redmine.org/releases/redmine-3.4.5.tar.gz
    

### 安装Ruby环境

参考官网说明，注意redmine版本对ruby版本的要求。 

1.下载地址：http://ftp.ruby-lang.org/pub/ruby/   
    
    //下载
    zmt@zmt-Lenovo:~/Desktop/work/tools$ wget http://ftp.ruby-lang.org/pub/ruby/2.3/ruby-2.3.1.tar.gz
    zmt@zmt-Lenovo:~/Desktop/work/tools$ tar -zxvf ruby-2.3.1.tar.gz 
    
    //查询openssl安装路径
    zmt@zmt-Lenovo:~$ openssl version -a
    OpenSSL 1.1.0g  2 Nov 2017
    built on: reproducible build, date unspecified
    platform: debian-amd64
    compiler: gcc -DDSO_DLFCN -DHAVE_DLFCN_H -DNDEBUG -DOPENSSL_THREADS -DOPENSSL_NO_STATIC_ENGINE -DOPENSSL_PIC -DOPENSSL_IA32_SSE2 -DOPENSSL_BN_ASM_MONT -DOPENSSL_BN_ASM_MONT5 -DOPENSSL_BN_ASM_GF2m -DSHA1_ASM -DSHA256_ASM -DSHA512_ASM -DRC4_ASM -DMD5_ASM -DAES_ASM -DVPAES_ASM -DBSAES_ASM -DGHASH_ASM -DECP_NISTZ256_ASM -DPADLOCK_ASM -DPOLY1305_ASM -DOPENSSLDIR="\"/usr/lib/ssl\"" -DENGINESDIR="\"/usr/lib/x86_64-linux-gnu/engines-1.1\"" 
    OPENSSLDIR: "/usr/lib/ssl"
    ENGINESDIR: "/usr/lib/x86_64-linux-gnu/engines-1.1"
    
    //看上面结果，确定openssl安装dir为：/usr/lib/ssl
    
    //安装：
    $ cd ruby-2.3.1
    $ ./configure  --with-openssl-dir=/usr/lib/ssl
    $ make
    $ sudo make install
    
    
3.检查是否安装成功：    
    
    zmt@zmt-Lenovo:~$ ruby -v
    ruby 2.3.1p112 (2016-04-26 revision 54768) [x86_64-linux]

看到上面显示说明安装成功。

### 创建空的数据库，并初始化用户

一般数据库名为redmine,但是可以自己更改。

1.MySQL

创建数据库登录用户redmine，密码为redmine
    
    //mysql要求５.6或以上版本
    
    CREATE DATABASE redmine CHARACTER SET utf8mb4;
    CREATE USER 'redmine'@'localhost' IDENTIFIED BY 'my_password';
    GRANT ALL PRIVILEGES ON redmine.* TO 'redmine'@'localhost';   
    
    //刷新权限
    flush privileges;
    

## 数据库连接配置设置

解压下载的`redmine-3.4.5.tar.gz`， 并进入解压包内。 在`config`目录下，可以看到文件`database.yml.example`，复制该文件命名为`database.yml`，放在同一个目录`config`下。下面就可以编辑文件`database.yml`了。  

1.mysql的配置

- 默认端口3306

        production:
          adapter: mysql2
          database: redmine
          host: localhost
          username: redmine
          password: redmine

- 不是3306端口，如下配置：

        production:
          adapter: mysql2
          database: redmine
          host: localhost
          port: 3307
          username: redmine
          password: redmine
          
按照以上配置修改好，保存并退出。

## redmine运行相关依赖包安装

Redmine uses [Bundler](http://gembundler.com/) to manage gems dependencies.   

1.首先按照Bundler

    gem install bundler
    
报错误：

    zmt@zmt-Lenovo:~$ gem install bundler
    ERROR:  While executing gem ... (Gem::Exception)
        Unable to require openssl, install OpenSSL and rebuild ruby (preferred) or use non-HTTPS sources

意思是，要要求系统按照OpenSSL，然后按上面步骤重新编译安装ruby。

按照openssl：
    
    sudo apt-get install openssl
    
重新编译按照ruby后再执行`gem install bundler`。      

还是不行，可能是因为不是以root用户安装的缘故。 

_注：_  没关系，我们不通过网络安装，而是直接下载gem安装包安装。

2.安装RubyGems安装

如果第一步能执行成功，不需要另外安装rubygems，因为安装ruby的时候已经安装。 

网址：https://rubygems.org/

下载并安装：

    # wget https://rubygems.global.ssl.fastly.net/rubygems/rubygems-2.6.6.tgz
    # tar zxvf rubygems-2.6.6.tgz
    # cd rubygems-2.6.6.tgz
    # ruby setup.rb
    
    //显示版本好，说民安装成功
    zmt@zmt-Lenovo:~/Desktop/work/tools/rubygems-2.6.6$ gem -v
    2.6.6
                  
安装成功，重新执行步骤1，即安装Bundler，执行：

    gem install bundler

 
       