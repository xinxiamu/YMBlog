---
title: github使用问题收集
date: 2020-12-28 10:54:03
categories: github
tags:
---

好记性，不如烂笔头！下面记录使用github过程中，经常出现的问题以及其解决方案……

## github pages突然无法访问的问题

1.问题描述：

github page如`xinxiamu.github.io`,突然无法访问了。

2.问题出现原因：

听说是dns解析出问题了。

3.解决方案：

- DNS查询

打开网址：https://tool.chinaz.com/

查询结果如下：

{%asset_img a-1.png%}

4.配置hosts

按照上上面查询到dns对应ip地址，打开电脑系统上的`hosts`文件。

打开win系统下的`hosts`文件方法：

进入目录`C:\Windows\System32\drivers\etc`,如下操作

{%asset_img a-2.png%}

打开命令窗口，编辑命令`PS C:\Windows\System32\drivers\etc> notepad .\hosts`执行。

然后编辑`hosts`文件如下：

{%asset_img a-3.png%}

保存关闭，刷新浏览器，就可以正常访问了。






