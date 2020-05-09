---
title: git使用
date: 2018-09-06 18:08:06
categories: git
tags: git问题
---

本文介绍git的使用方法以及一些问题……

## 版本控制系统(GIT)分支管理规范

https://www.jianshu.com/p/c35b939c5270

## 回滚版本库
在`git commit`多次后，版本库又会记录，但是在某次提交当中，提交了错误的代码，或者过大的文件，这个时候，就要做撤销，回滚。

方式一：    
`git log`   #拿到版本库id    
`git reset 版本库id`

## 清除用户名密码

问题描述：

    xinxiamu@xinxiamu-PC MINGW64 /g/jnyl/src
    $ git clone http://47.115.39.97:8081/ott-platform-ui/ott-platform-all-ui.git
    Cloning into 'ott-platform-all-ui'...
    remote: HTTP Basic: Access denied
    fatal: Authentication failed for 'http://47.115.39.97:8081/ott-platform-ui/ott-platform-all-ui.git/'


解决方案：

    git config --system --unset credential.helper
    
之后再进行git操作时，弹出用户名密码窗口，输入即可
