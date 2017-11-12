---
title: linux常用命令使用收藏
date: 2017-10-14 11:26:18
categories: linux
tags: linux-command
---
收集linux系统实践过程常用的命令。方便查看！

## 1. 系统用户管理
### 1.1. 查看所有系统用户
`[root@izwz924c5ufaoooso1wswiz ~]# cat /etc/passwd`

### 1.2. 查看系统用户对应的UID
`[root@izwz924c5ufaoooso1wswiz ~]# cat /etc/group`

### 1.3. 添加系统用户
`> useradd zmt`  添加用户zmt
`> passwd zmt`   为用户zmt添加密码，输入密码即可

### 1.4. 删除系统用户
`> userdel -r xz`  加上-r参数，userdel会删除用户的HOME目录以及邮件目录

_警告_ 在有大量用户的环境中使用-r参数时要特别小心。你永远不知道用户是否在其HOME目
录下存放了其他用户或其他程序要使用的重要文件。记住，在删除用户的HOME目录之
前一定要检查清楚！

### 1.5 修改用户
表7-3 用户账户修改工具

| 命 令 | 描 述 |
| :---: | :----: |
| usermod  |  修改用户账户的字段，还可以指定主要组以及附加组的所属关系   |
| passwd   |  修改已有用户的密码                      |
| chpasswd |  从文件中读取登录名密码对，并更新密码             |
| chage    |  修改密码的过期日期                      |
| chfn     |  修改用户账户的备注信息                    |
| chsh     |  修改用户账户的默认登录shell               |

_1. usermod_
usermod命令是用户账户修改工具中最强大的一个。它能用来修改/etc/passwd文件中的大部分字段，只需用与想修改的字段对应的命令行参数就可以了。参数大部分跟useradd命令的参数一样（比如，-c修改备注字段，-e修改过期日期，-g修改默认的登录组）。除此之外，还有另外一些可能派上用场的选项。

> - -l修改用户账户的登录名。
- -L锁定账户，使用户无法登录。
- -p修改账户的密码。
- -U解除锁定，使用户能够登录。

-L选项尤其实用。它可以将账户锁定，使用户无法登录，同时无需删除账户和用户的数据。
要让账户恢复正常，只要用-U选项就行了。

_2. passwd和chpasswd_
改变用户密码的一个简便方法就是用passwd命令。

    # passwd test
    Changing password for user test.
    New UNIX password:
    Retype new UNIX password:
    passwd: all authentication tokens updated successfully.
    #
    
如果只用passwd命令，它会改你自己的密码。系统上的任何用户都能改自己的密码，但只
有root用户才有权限改别人的密码。
_-e选项能强制用户下次登录时修改密码。你可以先给用户设置一个简单的密码，之后再强制
在下次登录时改成他们能记住的更复杂的密码。_
如果需要为系统中的大量用户修改密码，chpasswd命令可以事半功倍。chpasswd命令能从
标准输入自动读取登录名和密码对（由冒号分割）列表，给密码加密，然后为用户账户设置。你
也可以用重定向命令来将含有userid:passwd对的文件重定向给该命令。

    # chpasswd < users.txt
    #    
    
## 2. 系统用户组管理    

### 2.1 查看所有用户组
> `> cat /etc/group`
root:x:0:root
bin:x:1:root,bin,daemon
daemon:x:2:root,bin,daemon
sys:x:3:root,bin,adm
adm:x:4:root,adm,daemon
rich:x:500:
mama:x:501:
katie:x:502:
jessica:x:503:
mysql:x:27:
test:x:504:

和UID一样，GID在分配时也采用了特定的格式。系统账户用的组通常会分配低于500的GID
值，而用户组的GID则会从500开始分配。/etc/group文件有4个字段：

- 组名
- 组密码
- GID
- 属于该组的用户列表

### 2.2 创建新组
    
    [root@izwz924c5ufaoooso1wswiz ~]# groupadd spcs
    [root@izwz924c5ufaoooso1wswiz ~]# tail /etc/group
    mysql:x:1000:
    cgred:x:994:
    docker:x:993:
    nexus:x:1001:
    git:x:1002:
    elsearch:x:1003:
    epmd:x:992:
    rabbitmq:x:991:
    zmt:x:1004:
    spcs:x:1005:

_为spcs组添加成员_
`[root@izwz924c5ufaoooso1wswiz ~]# usermod -G spcs zmt`
`[root@izwz924c5ufaoooso1wswiz ~]# usermod -G spcs git`
两个系统用户zmt、git将添加到用户组spcs。

_说明_ 如果更改了已登录系统账户所属的用户组，该用户必须登出系统后再登录，组关系的更
改才能生效。

### 2.3 修改组
在/etc/group文件中可以看到，需要修改的组信息并不多。groupmod命令可以修改已有组的
GID（加-g选项）或组名（加-n选项）。

1. 修改组名

        # /usr/sbin/groupmod -n spcs spcselling
        # tail /etc/group
        haldaemon:x:68:
        xfs:x:43:
        gdm:x:42:
        rich:x:500:
        mama:x:501:
        katie:x:502:
        jessica:x:503:
        mysql:x:27:
        test:x:504:
        sharing:x:505:test,rich
        #
    
修改组名时，GID和组成员不会变，只有组名改变。由于所有的安全权限都是基于GID的，
你可以随意改变组名而不会影响文件的安全性。

## 3. 文件权限管理    

## 4. 开机启动命令-chkconfig


