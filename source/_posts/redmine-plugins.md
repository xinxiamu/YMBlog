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

    
        