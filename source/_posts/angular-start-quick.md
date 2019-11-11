---
title: angular快速上手
date: 2019-09-01 21:51:09
categories: angular
tags:
---

## 创建应用

打开某个需要存放项目的目录。在命令窗口执行：

    ng new angular-quick
    
用idea工具打开项目：

{% asset_img a-1.png %}  

## 创建组件模块

命令：`ng generate component 模块名称`

    E:\angular\study\angular-quick>ng generate component product-list
    
    E:\angular\study\angular-quick>"node"  "E:\angular\study\angular-quick\node_modules\.bin\\..\_@angular_cli@8.0.6@@angular\cli\bin\ng" generate component product-list
    CREATE src/app/product-list/product-list.component.html (27 bytes)
    CREATE src/app/product-list/product-list.component.spec.ts (664 bytes)
    CREATE src/app/product-list/product-list.component.ts (292 bytes)
    CREATE src/app/product-list/product-list.component.css (0 bytes)
    UPDATE src/app/app.module.ts (497 bytes)
    
    E:\angular\study\angular-quick>ng generate component top-bar

## 运行应用

    cd my-app
    ng serve --open
    
ng serve 命令会启动开发服务器、监视文件，并在这些文件发生更改时重建应用。

--open（或者只用 -o 缩写）选项会自动打开你的浏览器，并访问 http://localhost:4200/。    



    
    