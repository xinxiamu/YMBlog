---
title: gitlab服务器搭建（Centos7系统）
date: 2018-12-11 11:35:15
categories: gitlab
tags:
---

参考网址：https://about.gitlab.com/install/#centos-7

## 快速安装gitlab

1.安装相关依赖包：

基础依赖：

    sudo yum install -y curl policycoreutils-python openssh-server
    sudo systemctl enable sshd
    sudo systemctl start sshd
    sudo firewall-cmd --permanent --add-service=http
    sudo systemctl reload firewalld

发送邮件依赖：

    sudo yum install postfix
    sudo systemctl enable postfix
    sudo systemctl start postfix

2.添加gitlab仓库uri并安装：

添加安装包地址：

安装包地址：https://packages.gitlab.com/gitlab/gitlab-ce

    curl -s https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash
    
 安装：
 
    sudo EXTERNAL_URL="http://gitlab.ymu.com" yum install -y gitlab-ce
    
## 安装包rpm方式：

#### 1.和上面一样，安装依赖包。

#### 2.下载安装包：

1.社区版本地址：https://packages.gitlab.com/gitlab/gitlab-ce/packages/el/7/gitlab-ce-11.3.12-ce.0.el7.x86_64.rpm/download.rpm

说明：
```text
EL是Red Hat Enterprise Linux的简写 
- EL6软件包用于在Red Hat 6.x, CentOS 6.x, and CloudLinux 6.x进行安装 
- EL5软件包用于在Red Hat 5.x, CentOS 5.x, CloudLinux 5.x的安装 
- EL7 软件包用于在Red Hat 7.x, CentOS 7.x, and CloudLinux 7.x的安装
```

2.执行安装命令：

    [root@xr-server vagrant]# rpm -ivh gitlab-ce-11.3.12-ce.0.el7.x86_64.rpm 
    warning: gitlab-ce-11.3.12-ce.0.el7.x86_64.rpm: Header V4 RSA/SHA1 Signature, key ID f27eab47: NOKEY
    Preparing...                          ################################# [100%]
    Updating / installing...
       1:gitlab-ce-11.3.12-ce.0.el7       ################################# [100%]
    It looks like GitLab has not been configured yet; skipping the upgrade script.
    
           *.                  *.
          ***                 ***
         *****               *****
        .******             *******
        ********            ********
       ,,,,,,,,,***********,,,,,,,,,
      ,,,,,,,,,,,*********,,,,,,,,,,,
      .,,,,,,,,,,,*******,,,,,,,,,,,,
          ,,,,,,,,,*****,,,,,,,,,.
             ,,,,,,,****,,,,,,
                .,,,***,,,,
                    ,*,.
      
    
    
         _______ __  __          __
        / ____(_) /_/ /   ____ _/ /_
       / / __/ / __/ /   / __ `/ __ \
      / /_/ / / /_/ /___/ /_/ / /_/ /
      \____/_/\__/_____/\__,_/_.___/
      
    
    Thank you for installing GitLab!
    GitLab was unable to detect a valid hostname for your instance.
    Please configure a URL for your GitLab instance by setting `external_url`
    configuration in /etc/gitlab/gitlab.rb file.
    Then, you can start your GitLab instance by running the following command:
      sudo gitlab-ctl reconfigure
    
    For a comprehensive list of configuration options please see the Omnibus GitLab readme
    https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/README.md

    
3.修改Gitlab访问URL配置:

可以使用自定义域名，也可以直接IP地址+端口访问。

编辑文件：

    [root@xr-server ~]# cd /etc/gitlab/
    [root@xr-server gitlab]# ls
    gitlab.rb
    [root@xr-server gitlab]# vim gitlab.rb
    
修改：

    #external_url 'http://gitlab.example.com'
    external_url 'http://192.168.10.31:8080'

这样就以ip+端口方式访问。

4.重置并启动Gitlab

    sudo gitlab-ctl reconfigure
    sudo gitlab-ctl start

5.停止

    sudo gitlab-ctl stop
    
6.重启

    sudo gitlab-ctl restart    
        
    
   