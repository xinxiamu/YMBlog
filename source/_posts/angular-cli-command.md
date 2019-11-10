---
title: angular常用命令
date: 2019-11-10 10:37:48
categories: angular
tags:
---

本章介绍angular项目开发过程中经常用到的angular-cli命令。对这些常用命令要有个基本的认识以及基本的使用能力。

## 创建项目

    ng new project-name
    
该命令会在当前目录下，按照angular项目脚手架创建一个angular初始化项目。

## 运行应用

    cd project-name
    ng serve --open 或者 ng serve -o

执行以上命令，将会编译项目并启动，启动完自动跳到浏览器打开（命令参数`--open`）。   
 
默认的端口是`4200`,你也可以指定运行端口。`ng serve --port 4222 --open`。 

## 添加模块组件

    ng generate componet 组件名称
    
会在项目app目录中添加一个组件模块。包括四个文件。css样式文件，html模板文件，ts脚本文件，还有一个spec.ts的测试文件。

## 添加服务组件


             