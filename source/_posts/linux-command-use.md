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

## 5. 文档操作
### 5.1 复制文件-cp命令使用
1. 文件到文件复制
> 将文档a复制成b（相当于备份并改名）。
cp -i a b
或，
cp a b

2. 文件到目录复制
>将文档 file1复制到dir1目录下，复制后名称仍未file1
cp -i file1 dir1
或，
cp file1 dir1

3. 目录到目录复制
>将目录dir1复制到dir2目录下，复制结果目录被改名为dir2
cp -r dir1 dir2
将目录dir1下所有文件包括文件夹，都复制到dir2目录下
cp -r dir1/*.* dir2
常见错误：
1、提示cp: omitting directory错误
复制目录时，使用-r选项即可递归拷贝，如下：
cp -r dir1 dir2

## 6. 解压/压缩
### 6.1 压缩
> 压缩
  tar –cvf jpg.tar *.jpg //将目录里所有jpg文件打包成tar.jpg
  tar –czf jpg.tar.gz *.jpg   //将目录里所有jpg文件打包成jpg.tar后，并且将其用gzip压缩，生成一个gzip压缩过的包，命名为jpg.tar.gz
  tar –cjf jpg.tar.bz2 *.jpg //将目录里所有jpg文件打包成jpg.tar后，并且将其用bzip2压缩，生成一个bzip2压缩过的包，命名为jpg.tar.bz2
  tar –cZf jpg.tar.Z *.jpg   //将目录里所有jpg文件打包成jpg.tar后，并且将其用compress压缩，生成一个umcompress压缩过的包，命名为jpg.tar.Z
  rar a jpg.rar *.jpg //rar格式的压缩，需要先下载rar for linux
  zip jpg.zip *.jpg //zip格式的压缩，需要先下载zip for linux
  
### 6.2 解压
> 解压
  tar –xvf file.tar //解压 tar包
  tar -xzvf file.tar.gz //解压tar.gz
  tar -xjvf file.tar.bz2   //解压 tar.bz2
  tar –xZvf file.tar.Z   //解压tar.Z
  unrar e file.rar //解压rar
  unzip file.zip //解压zip
  
### 6.3 ubuntu 下rar解压工具安装方法：
 
> 1、压缩功能
 安装 sudo apt-get install rar
 卸载 sudo apt-get remove rar
 2、解压功能
 安装 sudo apt-get install unrar
 卸载 sudo apt-get remove unrar
 压缩解压缩.rar
 解压：rar x FileName.rar
 压缩：rar a FileName.rar DirName  
  
### 6.4 总结
> 1、`*.tar` 用 tar –xvf 解压
  2、`*.gz` 用 gzip -d或者gunzip 解压
  3、`*.tar.gz`和`*.tgz` 用 tar –xzf 解压
  4、`*.bz2` 用 bzip2 -d或者用bunzip2 解压
  5、`*.tar.bz2`用tar –xjf 解压
  6、`*.Z` 用 uncompress 解压
  7、`*.tar.Z` 用tar –xZf 解压
  8、`*.rar` 用 unrar e解压
  9、`*.zip` 用 unzip 解压    


## 配置环境变量

例子： 

    ########  JAVA_HOME #######
    JAVA_HOME=/server/java/jdk
    export JAVA_HOME
    PATH=$JAVA_HOME/bin:$JAVA_HOME/include:$JAVA_HOME/include/linux:$PATH
    export PATH
    CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
    export CLASSPATH
    
    
    ########  MySQL #######
    MYSQL_HOME=/server/databases/mysql-3810
    export MYSQL_HOME
    PATH=$MYSQL_HOME/bin:$PATH
    export PATH
    
##  SELinux 使用  

https://blog.csdn.net/yanjun821126/article/details/80828908     

#### selinux 开启和关闭

1.查看开启状态

    /usr/sbin/sestatus -v      ##如果SELinux status参数为enabled即为开启状态   
    
    -------------------------------
    
    [root@xr-server selinux]# /usr/sbin/sestatus -v 
    SELinux status:                 enabled
    SELinuxfs mount:                /sys/fs/selinux
    SELinux root directory:         /etc/selinux
    Loaded policy name:             targeted
    Current mode:                   enforcing
    Mode from config file:          enforcing
    Policy MLS status:              enabled
    Policy deny_unknown status:     allowed
    Max kernel policy version:      31
    
    Process contexts:
    Current context:                unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
    Init context:                   system_u:system_r:init_t:s0
    /usr/sbin/sshd                  system_u:system_r:sshd_t:s0-s0:c0.c1023
    
    File contexts:
    Controlling terminal:           unconfined_u:object_r:user_devpts_t:s0
    /etc/passwd                     system_u:object_r:passwd_file_t:s0
    /etc/shadow                     system_u:object_r:shadow_t:s0
    /bin/bash                       system_u:object_r:shell_exec_t:s0
    /bin/login                      system_u:object_r:login_exec_t:s0
    /bin/sh                         system_u:object_r:bin_t:s0 -> system_u:object_r:shell_exec_t:s0
    /sbin/agetty                    system_u:object_r:getty_exec_t:s0
    /sbin/init                      system_u:object_r:bin_t:s0 -> system_u:object_r:init_exec_t:s0
    /usr/sbin/sshd                  system_u:object_r:sshd_exec_t:s0

也可以用下面命令查看：

    [root@xr-server selinux]# getenforce 
    Enforcing
    
#### 关闭


1.临时关闭（不需要重启系统）

    setenforce 0  ##设置SELinux 成为permissive模式
    
    -----------------------------------------------------
    setenforce 1  ##设置SELinux 成为enforcing模式（开启）
    
2.修改配置文件关闭（需要重启系统）

修改/etc/selinux/config 文件

将SELINUX=enforcing改为SELINUX=disabled

然后重启系统即可。

## 免密码切换到root用户

 Linux下普通用户切换到root用户下，默认情况是需要输入密码，这在自动化脚本里面很不方便，因此需要实现普通用户免密切换到root用户。

解决方案：

- 以root用户登录shell终端，执行vim /etc/sudoers命令，找到如下图所示位置：   

      
- 在下方添加一行类似的数据，例如用户名称为elk,则添加内容为：
  
        ## Allow root to run any commands anywhere 
        root	ALL=(ALL) 	ALL
        vagrant    ALL=(ALL)       ALL
    
vagrant是用户名。在vagrant下，执行sudo -s就可以直接切换到root了。其它的账号类似。

           
- 保存之后，在普通用户下输入sudo -s命令就可以直接免密切换到root账户了。       
    
            