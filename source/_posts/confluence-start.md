---
title: confluence入门
date: 2021-12-20 11:44:42
categories: Confluence
tags:
---

Confluence是一个强大的知识管理系统……

[官网](https://www.atlassian.com/zh/software/confluence)

## 先安装jira

## 安装、破解

https://blog.whsir.com/post-5854.html

## 卸载

1.进入安装目录
```shell
[root@xr-server-dev ~]# cd /opt/atlassian/confluence/
[root@xr-server-dev confluence]# ls
bin           conf        CONTRIBUTING.md  jre  LICENSE   logs    README.html  README.txt     RUNNING.txt      temp       webapps
BUILDING.txt  confluence  install.reg      lib  licenses  NOTICE  README.md    RELEASE-NOTES  synchrony-proxy  uninstall  work
[root@xr-server-dev confluence]# clear
[root@xr-server-dev confluence]# ls
bin           conf        CONTRIBUTING.md  jre  LICENSE   logs    README.html  README.txt     RUNNING.txt      temp       webapps
BUILDING.txt  confluence  install.reg      lib  licenses  NOTICE  README.md    RELEASE-NOTES  synchrony-proxy  uninstall  work
[root@xr-server-dev confluence]# ./uninstall 
Are you sure you want to completely remove Confluence 7.4.6 and all of its components?
Yes [y, Enter], No [n]
y
Uninstalling Confluence 7.4.6 ...
Confluence 7.4.6 was successfully removed from your computer.
Finishing uninstallation ...

```

2.进入数据目录 /var/atlassian 删除里面的文件

3.进入 /etc/init.d/ 删除多余的 confluence 开机启动项

4.下面即可重新安装！！！
