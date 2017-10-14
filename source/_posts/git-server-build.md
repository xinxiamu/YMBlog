---
title: git服务器使用
date: 2017-10-14 09:58:47
categories: git
tags: git-server-build
---
搭建git服务器之前，要在服务器上安装git环境。这里假定已经安装git环境。
本篇主要有两部分内容，一、git服务器搭建、二、用户的分配以及用户对文件权限的控制。

首先安装最新版本git

一、创建git用户，用来管理git服务，为git设置密码
> id git
查看是否已经有该用户，如果没有则创建用户
> useradd zmt
> passwd **** #设置zmt用户密码，注意记得密码

二、创建git仓库
> mkdir -p /server/data/git/test.git
> git init --bare /server/data/git/test.git/ #初始化空的版本库于test.git
> cd /server/data/git/
> chown -R zmt:gits test.git/ #把仓库的owner设置为gits用户组下zmt系统用户

三、客户端克隆仓库
>  git clone git@119.23.78.160:/server/data/git/test.git
Cloning into 'test'...
The authenticity of host '119.23.78.160 (119.23.78.160)' can't be established.
ECDSA key fingerprint is SHA256:u7IEulSBpZOfmqBXkr8tW4JJ423qbuM7kMERgAw6MMk.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '119.23.78.160' (ECDSA) to the list of known hosts.
git@119.23.78.160's password:

#输入git系统用户密码：a1234567
warning: You appear to have cloned an empty repository.