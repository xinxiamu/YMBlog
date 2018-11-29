---
title: wkhtmltopdf使用记录
date: 2018-06-19 18:32:21
categories: pdf
tags: wkhtmltopdf
---

html转pdf工具wkhtmltopdf的使用记录……

官网：https://wkhtmltopdf.org/

使用文档：   
https://wkhtmltopdf.org/usage/wkhtmltopdf.txt
https://www.cnblogs.com/colder/p/5819197.html

## centos下安装环境

1.依赖包安装： `yum install zlib fontconfig freetype libX11 libXext libXrender` 


## 使用问题收集

1.中文字体乱码的问题。

linux：把字体{%asset_link simsun.ttc simsun.ttc%}添加到系统`usr/share/fonts`下。

2.设定纸张大小，内容随纸张大小。

3.Arial字体

直接把{%asset_link Arial.zip Arial%}中相关字体文件添加到系统`usr/share/fonts`。

参考：https://blog.csdn.net/churujianghu/article/details/75076255

示例：   
  
    ./wkhtmltopdf --page-height 5cm --page-width 7cm --margin-bottom 0cm --margin-top 0.1cm --margin-left 0.1cm --margin-right 0cm --disable-smart-shrinking   label.html label.html label.html label.pdf

## 页眉处理
    
样例1：
    
    wkhtmltopdf.exe --footer-left "cxmr(500)        制表日期：[date] [time]"  --footer-right "制表者： XC-TEST001        页次： [page]/[topage]"  --footer-line  orders-atn.html a.pdf

样例2：

    wkhtmltopdf.exe --footer-left "cxmr(500)        制表日期：[date] [time]"  --footer-right "制表者： XC-TEST001        页次： [page]/[topage]"  --footer-line --footer-font-size 8  --footer-spacing 5 orders-atn.html b.pdf
    
样例3：
    
    wkhtmltopdf.exe --footer-left "cxmr(500)        制表日期：[date] [time]"  --footer-right "制表者： XC-TEST001        页次： [page]/[topage]"  --footer-line --footer-font-size 8  --footer-spacing 5 orders-atn.html orders-atn.html orders-atn.html b.pdf    