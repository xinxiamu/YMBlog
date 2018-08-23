---
title: 项目管理系统Redmine使用
date: 2018-06-01 21:35:04
categories: redmine
tags: 项目管理和bug跟踪系统redmine
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
    zmt@zmt-Lenovo:~/Desktop/work/tools$ wget http://ftp.ruby-lang.org/pub/ruby/2.4/ruby-2.4.4.tar.gz
    zmt@zmt-Lenovo:~/Desktop/work/tools$ tar -zxvf ruby-2.4.4.tar.gz 
    
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
    $ cd ruby-2.4.4
    $ ./configure  --with-openssl-dir=/usr/lib/ssl
    $ make
    $ sudo make install
    
    
3.检查是否安装成功：    
    
    zmt@zmt-Lenovo:~$ ruby -v
    ruby 2.4.4p296 (2018-03-28 revision 63013) [x86_64-linux]

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
    

### 数据库连接配置设置

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

### redmine运行相关依赖包安装

Redmine uses [Bundler](http://gembundler.com/) to manage gems dependencies.   

1.首先按照Bundler

    sudo gem install bundler
    
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


    安装RubyGems安装
    
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

 
2.然后可以利用bundler安装redmine相关依赖包

Then you can install all the gems required by Redmine using the following command:

    bundle install --without development test
    
<<----未完--->>

## Centos7下安装全过程记录

参考网址：http://www.redmine.org/projects/redmine/wiki/RedmineInstall

应用版本信息：
redmine： redmine-3.4.6.tar.gz   
ruby： ruby-2.3.6.tar.gz

首先安装系统相关包：

    yum -y install patch make gcc gcc-c++ gcc-g77 flex* bison file  
    yum -y install libtool libtool-libs libtool-ltdl-devel* autoconf kernel-devel automake libmcrypt*  
    yum -y install libjpeg libjpeg-devel libpng libpng-devel libpng10 libpng10-devel gd gd-devel  
    yum -y install freetype freetype-devel libxml2 libxml2-devel zlib zlib-devel  
    yum -y install glib2 glib2-devel bzip2 bzip2-devel libevent libevent-devel  
    yum -y install ncurses ncurses-devel curl curl-devel e2fsprogs  
    yum -y install e2fsprogs-devel krb5 krb5-devel libidn libidn-devel  
    yum -y install openssl openssl-devel vim-minimal nano sendmail  
    yum -y install fonts-chinese gettext gettext-devel  
    yum -y install gmp-devel pspell-devel   
    yum -y install readline* libxslt* pcre* net-snmp* gmp* libtidy*  
    yum -y install ImageMagick* subversion*  

### 下载redmine安装包

可以在官网下载正式发布的二进制包[下载](http://www.redmine.org/projects/redmine/wiki/Download)。 

### 创建空的数据库以及相关数据库用户

首先要保证已经安装好数据库。下面以mysql为例进行安装。mysql版本>5.5.2。     
创建脚本：    
   
    CREATE DATABASE redmine CHARACTER SET utf8mb4;
    #CREATE USER 'redmine'@'localhost' IDENTIFIED BY 'my_password'; #这样的，navicat客户端无法登录
    CREATE USER 'redmine'@'localhost' IDENTIFIED WITH mysql_native_password BY 'my_password';
    GRANT ALL PRIVILEGES ON redmine.* TO 'redmine'@'localhost';
    
### 为redmine配置数据库

首先，解压redmine包，并进入config目录。  

`[root@ymu config]# cp database.yml.example database.yml`

然后编辑`database.yml`：

    production:
      adapter: mysql2
      database: redmine
      host: localhost
      port: 3307
      username: redmine
      password: redmine
      
### 安装依赖包

1.安装ruby环境。

- 安装上面描述方式，ruby源码包安装方式。选择版本安装。
- 系统源安装。`yum install gem`。会自动安装ruby环境。

下面采用源码编译安装方式。   
检查是否安装成功：   
`ruby -v`   `gem -v`

问题解决：安装完ruby却提示`[/usr/bin/ruby: No such file or directory]`
    
`ln -s /usr/local/bin/ruby /usr/bin/ruby`   
`ln -s /usr/local/bin/gem /usr/bin/gem` 
 

2.安装`bundler`

redmine的依赖包都通过bundler来安装。   

`gem install bundler`

3.安装所有依赖

执行下面命令前，记得要按照上面方法安装所有系统依赖包。

    [root@ymu ~]# cd /server/tools/redmine-3.4.6/
    [root@ymu redmine-3.4.6]# bundle install --without development test
   

### 安全生成存储会话

`bundle exec rake generate_secret_token`
 

### 创建表

    [root@ymu redmine-3.4.6]# bundle exec rake generate_secret_token
    [root@ymu redmine-3.4.6]# RAILS_ENV=production bundle exec rake db:migrate

查看数据库，可以看到已经创建了很多表。     

### 设置数据库默认数据

    [root@ymu redmine-3.4.6]# RAILS_ENV=production bundle exec rake redmine:load_default_data
    
    Select language: ar, az, bg, bs, ca, cs, da, de, el, en, en-GB, es, es-PA, et, eu, fa, fi, fr, gl, he, hr, hu, id, it, ja, ko, lt, lv, mk, mn, nl, no, pl, pt, pt-BR, ro, ru, sk, sl, sq, sr, sr-YU, sv, th, tr, uk, vi, zh, zh-TW [en] zh
    ====================================
    Default configuration data loaded.

记得：输入zh，然后按下回车。否则会是英文版本数据。

### 设置文件权限

- 如果上面的所有操作都是root用户，那么就不必要设置文件权限，可以跳过此步骤。 （_上面以root操作，跳过这步骤_）    
- 如果不是在root超级用户下，则要设置文件权限，否则redmine应用程序无法操作一些文件权限。    

在redmine解压根目录下，这些文件必须赋予权限：  

    files (storage of attachments)
    log (application log file production.log)
    tmp and tmp/pdf (create these ones if not present, used to generate PDF documents among other things)
    public/plugin_assets (assets of plugins)
    
如果没有这些文件：     
    
    mkdir -p tmp tmp/pdf public/plugin_assets
    sudo chown -R redmine:redmine files log tmp public/plugin_assets
    sudo chmod -R 755 files log tmp public/plugin_assets
    
如果都有：   

     sudo chown -R redmine:redmine files log tmp public/plugin_assets
     sudo chmod -R 755 files log tmp public/plugin_assets
 
_注意_:保证下面目录不包含可执行文件。
 
    sudo find files log tmp public/plugin_assets -type f -exec chmod -x {} +    
    
### 测试是否安装成功

按照下面经验执行操作：

    bundle exec rails server webrick -e production   //在redmine安装目录下执行
    
    //注意，服务器端执行上面命令可能报错，可能是端口不可用，可以改变端口：
    bundle exec rails server webrick -e production -p 8889
    
    改变端口执行服务成功，但是在客户机子不能访问，此时要这么执行：
     bundle exec rails server webrick -e production -b 0.0.0.0 -p 8889
    
    ok搞定
    
    守护进程模式执行：
     nohup bundle exec rails server webrick -e production -b 0.0.0.0 -p 8889 &
    或者：
    bundle exec rails server webrick -e production -b 0.0.0.0 -p 8889 -d    
    
然后就可以访问：http：//loaclhost:port。  

### 登录redmine

默认账号密码： 
username=admin  
pwd=admin

打开网址后，用默认账号密码登录后，会要求修改密码。把密码改为： 
username=admin  
pwd=admin123    

按页面提示，填写修改相关信息即可。   

### 修改配置（不修改则采用默认的配置）

redmine的配置设置文件放在：config/configuration.yml。  
如果要自己定义配置，则可以`copy config/configuration.yml.example to config/configuration.yml`，然后编辑`configuration.yml`文件即可。 

记得：修改配置文件后重启redmine，否则不生效。  

### 重启redmine

采用直接kill掉：

    lsof -i:3000
    kill -9 pid

### 设置Email/SMTP服务器

项目管理中，分配了任务或者测试提交了bug给某个开发人员，那么可以通过邮件及时的提醒他。    

下面我们就来配置邮件服务器：  
参考：http://www.redmine.org/projects/redmine/wiki/EmailConfiguration  

1.编辑配置文件：config/configuration.yml

添加邮箱服务器配置，异步发送邮件（qq邮箱为例子）：

    email_delivery:
        delivery_method: :async_smtp
        async_smtp_settings:
          address: "smtp.qq.com"
          port: 25
          authentication: :login
          domain: 'qq.com'
          user_name: '932852117@qq.com'
          password: '××××××'
              
下面是实际操作内容：              

    # default configuration options for all environments
    default:
      # Outgoing emails configuration
      # See the examples below and the Rails guide for more configuration options:
      # http://guides.rubyonrails.org/action_mailer_basics.html#action-mailer-configuration
      email_delivery:
        delivery_method: :async_smtp
        async_smtp_settings:
          address: "smtp.qq.com"
          port: 25
          authentication: :login
          domain: 'qq.com'
          user_name: '932852117@qq.com'
          password: '××××××'
    
      # ==== Simple SMTP server at localhost

2.开启邮箱服务器

{%asset_img b.png%}

3.重启redmine并测试是否配置成功。 
        
登录redmine，在管理->配置中： 

{%asset_img c.png%}

输入配置的邮箱地址后，保存。然后点击右下角的`发送测试邮件`：     

{%asset_img d.png%}

看到绿色提示`邮件已发送至 zhangmutian@xcsqjr.com`。证明配置已成功，可以愉快的使用了。



------------------------------------------------------

恭喜恭喜，到此，你已成功安装redmine了，并且它已经具备了该有的基础功能了。赶紧来看下它帅帅的样子吧：

{%asset_img a.png%}

尽情的去探索redmine很多很多，酷酷的特性吧，让它正真成为你在项目管理中的瑞士军刀……





          