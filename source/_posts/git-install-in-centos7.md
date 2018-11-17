---
title: git在centos7下源码编译安装
date: 2017-10-14 10:04:31
categories: git
tags: git-install-in-centos
---

## 一、安装依赖包
yum -y install zlib-devel curl-devel openssl-devel perl cpio expat-devel gettext-devel openssl zlib autoconf tk perl-ExtUtils-MakeMaker

## 二、下载最新稳定版本安装包
源码网址：https://github.com/git/git

## 三、查看是否已经安装了旧版本
> git --version
如果有显示版本信息，则先卸载旧版本
> yum -y remove git
> yum autoremove

## 四、解压安装包,并安装
> cd /server/tools
> unzip git-2.14.1.zip
> cd git-2.14.1
> make prefix=/server/git all   #安装在目录/server/git下
>  make prefix=/server/git install

## 五、添加link
> ln -s /server/git/bin/git /usr/bin/
注：这一步对于原本系统中有旧版git的系统很重要，会报告Link已存在，此时要删除原来的Link即/usr/bin/git，再执行第六步。

## 六、将git设置为默认路径，不然后面克隆时会报错
>  ln -s /server/git/bin/ git-upload-pack /usr/bin/git-upload-pack
>  ln -s /server/git/bin/git-receive-pack /usr/bin/git-receive-pack

## 七、查看版本
> git --version #两个横杆

-----------------------------------------------------

_更新git版本_
下载最新源码重新编译覆盖即可

----------------------------------------- 官网 -------------------------------------
从源代码安装
有人觉得从源码安装 Git 更实用，因为你能得到最新的版本。 二进制安装程序倾向于有一些滞后，当然近几年 Git 已经成熟，这个差异不再显著。
如果你想从源码安装 Git，需要安装 Git 依赖的库：curl、zlib、openssl、expat，还有libiconv。 如果你的系统上有 yum （如 Fedora）或者 apt-get（如基于 Debian 的系统），可以使用以下命令之一来安装最小化的依赖包来编译和安装 Git 的二进制版：

    $ sudo yum install curl-devel expat-devel gettext-devel \
    openssl-devel zlib-devel
    $ sudo apt-get install libcurl4-gnutls-dev libexpat1-dev gettext \
    libz-dev libssl-dev

为了能够添加更多格式的文档（如 doc, html, info），你需要安装以下的依赖包：

    $ sudo yum install asciidoc xmlto docbook2x
    $ sudo apt-get install asciidoc xmlto docbook2x

当你安装好所有的必要依赖，你可以继续从几个地方来取得最新发布版本的 tar 包。 你可以从 Kernel.org 网站获取，网址为 https://www.kernel.org/pub/software/scm/git，或从 GitHub 网站上的镜像来获得，网址为 https://github.com/git/git/releases。 通常在 GitHub 上的是最新版本，但 kernel.org 上包含有文件下载签名，如果你想验证下载正确性的话会用到。
接着，编译并安装：

    $ tar -zxf git-2.0.0.tar.gz
    $ cd git-2.0.0
    $ make configure
    $ ./configure --prefix=/usr
    $ make all doc info
    $ sudo make install install-doc install-html install-info

完成后，你可以使用 Git 来获取 Git 的升级：

    $ git clone git://git.kernel.org/pub/scm/git/git.git