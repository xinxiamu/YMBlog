---
title: centos免密码登录
date: 2017-11-30 14:53:47
categories: CentOs
tags: centos登录
---

配置免密码登录服务器。不用每次都要输入密码。

## 环境说明

客户机:Mac OS X
服务器:CentOS 7
客户端:OpenSSH,OS X及大多数Linux都内置了OpenSSH.’ssh -v’命令可以查看版本.

## 客户机配置

1. 查看~/.ssh文件夹,若已经存在有公钥文件(id_rsa.pub),私钥文件(id_rsa),则可以跳过客户端配置.  
2. 生成密钥文件.$ ssh-keygen 然后一路回车.然后~/.ssh下会生成id_rsa.pub和id_rsa, 其中id_rsa文件起到唯一标识你的客户机的作用.注意:不要改这两个文件的文件名,ssh登陆时会读取id_rsa文件.

## 服务器配置

1.修改sshd配置文件(/etc/ssh/sshd_config).  
找到以下内容，并去掉注释符”#“

> RSAAuthentication yes (我新购的机器Centos7.4的，无需配置这句)
PubkeyAuthentication yes
AuthorizedKeysFile  .ssh/authorized_keys

2.配置authorized_keys文件.若’~/.ssh/authorized_keys’不存在,则建立.ssh文件夹和authorized_keys文件.将上文中客户机id_rsa.pub的内容拷贝到authorized_keys中.PS:可以在客户机中执行命令来拷贝:

    cat ~/.ssh/id_rsa.pub | ssh user@host “cat - >> ~/.ssh/authorized_keys”
    
>注意:
1 .ssh目录的权限必须是700
2 .ssh/authorized_keys文件权限必须是600

重启ssh： service sshd restart   
然后客户先先执行：ssh -v user@host (-v 调试模式)会显示一些登陆信息.若登陆失败,或者仍然要输入密码,可以在服务器查看日志文件:/var/log/secure.若登陆成功,则以后就可以用’ssh user@host’ 直接登陆了,不用输入密码.

-----------------------------------------------------------
## 更简单方式

1、执行命令：ssh-keygen -t rsa -C "xx@qq.com"(随便编个字符串，一般用邮箱）
2、之后一路回车就行啦；会在～（home）目录下中产生.ssh（隐藏）文件夹；
3、里面有两个文件id_rsa(私钥)、id_rsa.pub(公钥)文件



yutao@localhost ~]$ ssh-copy-id yutao@192.168.161.132 #把秘钥拷贝到远程服务器
