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

## SSL验证无法连接仓库

问题：

```shell
14:54:32.297: [java-core] git -c credential.helper= -c core.quotepath=false -c log.showSignature=false push --progress --porcelain origin refs/heads/master:master
fatal: unable to access 'https://github.com/xinxiamu/java-core.git/': OpenSSL SSL_read: Connection was reset, errno 10054
```

解决：

执行下面命令，然后再继续后面命令即可：
```shell
git config --global http.sslVerify "false"
```

## git项目出现目录安全检查

问题：

Win10系统，使用VSCode提示错误fatal: detected dubious ownership in repository at

解决：

win10电脑打开`git bash`命令窗口执行：
```shell
git config --global --add safe.directory '*'
```
