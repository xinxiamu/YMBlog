---
title: angular入门
date: 2018-06-23 10:32:10
categories: angular
tags: angular入门
---

本文介绍如何搭建angular开发环境……

[中文官网](https://angular.cn)
[英文官网](https://angular.io)

## 环境安装

### ubuntu下

_nodejs、npm环境_

下载：https://nodejs.org/en/download/

`nodejs`自带`npm`。

- 现在二进制包，解压到相关目录。
- 配置`nodejs`环境变量。
    
    
    export NODE_HOME=~/Desktop/work/tools/node
    export PATH=${NODE_HOME}/bin:$PATH
    
- 检查安装是否成功


    zmt@zmt-Lenovo:~$ node -v
    v10.4.1
    zmt@zmt-Lenovo:~$ npm -v
    6.1.0

- 安装`npm`淘宝镜像

`npm install -g cnpm --registry=https://registry.npm.taobao.org`  


_angular-cli_ 安装

全局安装

`cnpm install -g @angular/cli`

### Windows环境下

直接下载nodejs的可执行文件，运行安装即可。安装成功后，打开命令行窗口检查是否安装成功。

`node -v`       
`npm -v`    

显示安装的版本号，则表明正常安装。

后面步骤同上……

## 跑起angular项目

1. 创建新项目

    `ng new my-app`

2. 启动服务器

    `cd my-app`
    
    `ng serve --open` 或者 `ng serve -o`
    
如果一切顺利，会自动打开浏览器显示网页。至此，一个`hello wold`项目完成了。    
