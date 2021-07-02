---
title: word文档转pdf
date: 2021-07-02 10:11:21
categories:
tags:
---

本文介绍几种word文件转pdf的方法。

## poi + itexpdf（或者wkhtmltopdf）-（免费）

思路是，利用`poi`把word转成html，然后再利用itexpdf把html转成pdf。  
或者利用wkhtmltopdf(需要安装，不友好)把html转成pdf。

发现问题：poi把word转成html会出现样式不正确，不准确的问题，所以导致后面转pdf并不完美。

所以不推荐使用。


## OpenOffice 转word为pdf（免费）

不好地方：需要安装OpenOffice软件，造成系统外部依赖。并且转换后的pdf也不是那么完美，有变样的情况。


## Spire.doc 商业软件

这是一家国内公司开发的商业组件，需要购买授权服务。

试用了免费版本，能够完美转换word为pdf。但是免费版本只能转换前三页，遗憾。

[网址](https://www.e-iceblue.cn/)

## aspose-words 商业软件

这是一家国外公司开发的商业组件。和Spir.doc类同。需要购买。

但是          

一通百度后，发现有牛人，把java-sdk的破解了。所以试了下，可以完美转换word成pdf。

商业使用的话，条件允许还是建议购买授权。以避免不必要的意料之外的损失。

[破解方法](https://blog.csdn.net/xxw19950701/article/details/115724571)

[网址](https://github.com/aspose)
