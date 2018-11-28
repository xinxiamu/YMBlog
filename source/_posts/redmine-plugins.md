---
title: redmine常见插件安装使用
date: 2018-08-24 16:04:35
categories: redmine
tags: redmine插件
---

redmine项目管理系统之所以强大并让大家喜欢，正是其插件化的管理模式。各种插件为其增添了不少上天入地的能力！     
因此，我们在这里就来介绍redmine的插件功能的使用，以及收集一些在开发管理中常用的插件功能。

## redmine插件的安装

1.查找插件并下载(官方仓库)：

官网地址：http://www.redmine.org/plugins?page=1

为了避免下载的插件版本和redmine（3.4.*）的版本冲突，必须选定对应版本下的插件下载。       
{%asset_img a.png%}         

2.其它途径下载，比如github上。

 
- 安装： 

下载插件到plugins目录下，然后执行`bundle exec rake redmine:plugins:migrate RAILS_ENV=production`，再然后重启即可。
    
- 卸载： 

先执行`bundle exec rake redmine:plugins:migrate NAME=plugin_name VERSION=0 RAILS_ENV=production`，然后删除plugins目录下相应的插件目录，重启就可以了。 


## 切图粘贴插件`redmine_image_clipboard_paste`(推荐)

下载网址：https://github.com/thorin/redmine_image_clipboard_paste    

这个是人家改过的新版本，兼容redmine`3.3.*`版本，旧的版本`redmine_image_clipboard_paste`不可用
1.安装：

    cd /path/to/redmine/
    git clone git://github.com/thorin/redmine_image_clipboard_paste.git plugins/redmine_image_clipboard_paste
    bundle exec rake redmine:plugins:migrate RAILS_ENV=production
    
2.卸载：
    
    cd /path/to/redmine/
    bundle exec rake redmine:plugins:migrate NAME=redmine_image_clipboard_paste VERSION=0 RAILS_ENV=production
    rm -rf plugins/redmine_image_clipboard_paste
    
## 方便查看问题中的图片`Lightbox Plugin 2`

在Lightbox更加方便的查看问题中的图片，那些小的图片，就不用再点击进去了，直接鼠标放上去就能看。 
    
    
## 代码审查插件`Code Review`

参考网址：http://www.redmine.org/plugins/redmine_code_review

代码审查插件允许对代码仓库中的代码进行审查批阅，做注释。有效地对项目成员写的代码质量做出把控。 

具体使用：

## 工时单插件

可以方便查看各个人的各个项目的工时情况

地址：http://www.redmine.org/plugins/timesheet

## 添加表情插件`Emoji Button`

添加几个表情，使枯燥的编程工作变得更加有趣！  

地址：http://www.redmine.org/plugins/redmine_emojibutton

## office文档查看插件

地址：https://www.redmine.org/plugins/redmine_preview_office

### 安装

实验环境：
系统：centos7 64位  
redmine：3.4.6   
ruby：2.3.6  
rails：4.2.8

#### 一.安装[libreoffice](https://www.libreoffice.org)

libreoffice提供命令把word文档，excel文档转成pdf格式等。这里同时提供了一个思路，开发应用时候，文档格式转换可以采用它。

下载下面安装包到服务器：

    wget https://downloadarchive.documentfoundation.org/libreoffice/old/5.3.6.1/rpm/x86_64/LibreOffice_5.3.6.1_Linux_x86-64_rpm_sdk.tar.gz
    wget https://downloadarchive.documentfoundation.org/libreoffice/old/5.3.6.1/rpm/x86_64/LibreOffice_5.3.6.1_Linux_x86-64_rpm_langpack_zh-CN.tar.gz
    wget https://downloadarchive.documentfoundation.org/libreoffice/old/5.3.6.1/rpm/x86_64/LibreOffice_5.3.6.1_Linux_x86-64_rpm.tar.gz    

    [root@iZwz9b0bqrkbhqd5lu2pwhZ LibreOffice]# ls
    LibreOffice_6.1.3.2_Linux_x86-64_rpm                 LibreOffice_6.1.3.2_Linux_x86-64_rpm_sdk                  LibreOffice_6.1.3_Linux_x86-64_rpm_sdk.tar.gz
    LibreOffice_6.1.3.2_Linux_x86-64_rpm_langpack_zh-CN  LibreOffice_6.1.3_Linux_x86-64_rpm_langpack_zh-CN.tar.gz  LibreOffice_6.1.3_Linux_x86-64_rpm.tar.gz

解压上面安装包，解压后，里面都有目录`RPMS`,安装里面的rpm包即可：   

    yum localinstall *.rpm
    
很顺利的安装成功。    
    
下面检查libreoffice是否可用：    

    把test.doc转换成html，保存在test目录
    libreoffice6.0 --invisible --convert-to html --outdir ./test test.doc 

彻底卸载libreoffice：    

    yum  erase libreoffice\*
 
#### 二.安装[redmine_preview_office](https://www.redmine.org/plugins/redmine_preview_office)插件
 
进入redmine安装的根目录：   
 
    # git clone https://github.com/HugoHasenbein/redmine_preview_office.git plugins/redmine_preview_office
    # bundle exec rake redmine:plugins:migrate RAILS_ENV=production 

然后重启redmine即可。如下图：  

{%asset_img b-1.png%}  

说明插件安装成功了。

卸载插件：

    [root@iZwz9b0bqrkbhqd5lu2pwhZ redmine-3.4.6]# bundle exec rake redmine:plugins:migrate NAME=redmine_preview_office VERSION=0 RAILS_ENV=production
    Migrating redmine_preview_office (Redmine Preview Office)...
    [root@iZwz9b0bqrkbhqd5lu2pwhZ redmine-3.4.6]# rm -rf plugins/redmine_preview_office/

重启redmine即可。

#### 三.直接浏览文档

点开一个问题的word文档，很遗憾，没能成功显示……

坑爹……待续


   
        