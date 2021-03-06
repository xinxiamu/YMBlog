---
title: angular入门
date: 2018-06-23 10:32:10
categories: angular
tags:
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

- 设置npm的镜像地址

1.查看默认

    > npm get registry
    
    http://registry.npmjs.org 
    
2.设置

    npm set registry https://registry.npm.taobao.org
    
3.再次查看
    
    > npm get registry
    
    https://registry.npm.taobao.org/        

可以看到，已经设置成淘宝的npm镜像了。这样，后面使用ng-cli的时候，默认的npm就是从淘宝镜像获取依赖了。

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


## 卸载并安装指定版本Angular CLI

1.卸载之前的版本

npm uninstall -g @angular/cli

2.清除缓存，确保卸载干净

npm cache clean --force

3.检查是否卸载干净

输入命令

ng -v

若显示command not found则卸载干净

4.安装指定版本

npm install -g @angular/cli@1.6.3

5.检查版本号

ng version


