---
title: ubuntu使用常用命令收集
date: 2018-03-01 16:02:21
categories: ubuntu
tags:
---

记录在使用ubuntu系统过程中常见命令……

## 查看占用端口

`netstat -ln|grep 8388`或者`lsof -i:8388`

## 关闭端口下应用

`kill -9 PID号`


## 防火墙

1.安装

`sudo apt-get install ufw` 

2.启用

    sudo ufw enable
    sudo ufw default deny
    运行以上两条命令后，开启了防火墙，并在系统启动时自动开启。

3.开启/禁用

    sudo ufw allow|deny [service]
    打开或关闭某个端口，例如：
    sudo ufw allow smtp　允许所有的外部IP访问本机的25/tcp (smtp)端口
    sudo ufw allow 22/tcp 允许所有的外部IP访问本机的22/tcp (ssh)端口
    sudo ufw allow 53 允许外部访问53端口(tcp/udp)
    sudo ufw allow from 192.168.1.100 允许此IP访问所有的本机端口
    sudo ufw allow proto udp 192.168.0.1 port 53 to 192.168.0.2 port 53
    sudo ufw deny smtp 禁止外部访问smtp服务
    sudo ufw delete allow smtp 删除上面建立的某条规则
    
4.查看防火墙状态

`sudo ufw status`    

开启/关闭防火墙 (默认设置是’disable’)

`# ufw enable|disable`

5.UFW 使用范例：

    允许 53 端口
    
    $ sudo ufw allow 53
    
    禁用 53 端口
    
    $ sudo ufw delete allow 53
    
    允许 80 端口
    
    $ sudo ufw allow 80/tcp
    
    禁用 80 端口
    
    $ sudo ufw delete allow 80/tcp
    
    允许 smtp 端口
    
    $ sudo ufw allow smtp
    
    删除 smtp 端口的许可
    
    $ sudo ufw delete allow smtp
    
    允许某特定 IP
    
    $ sudo ufw allow from 192.168.254.254
    
    删除上面的规则
    
    $ sudo ufw delete allow from 192.168.254.254

## Ubuntu Server更改系统时间

本地小型机，更改系统时间，尽可能在外有外网情况下确保时间准确。

1. 查看系统时间时区是否正确。

```shell
root@hgbio-server:~# hwclock 
2023-07-12 08:17:30.356828+00:00
```

查看是否是+08：00，即东八区。上面命令结果明显不是。

所以下面使用`tzselect`修改时区。

```shell
root@hgbio-server:~# tzselect 
Please identify a location so that time zone rules can be set correctly.
Please select a continent, ocean, "coord", or "TZ".
1) Africa							     7) Europe
2) Americas							     8) Indian Ocean
3) Antarctica							     9) Pacific Ocean
4) Asia								    10) coord - I want to use geographical coordinates.
5) Atlantic Ocean						    11) TZ - I want to specify the timezone using the Posix TZ format.
6) Australia
#? ^[[B^H4
Please enter a number in range.
#? 4
Please select a country whose clocks agree with yours.
1) Afghanistan		      9) Cambodia		  17) Hong Kong		       25) Kazakhstan		    33) Malaysia		 41) Qatar		      49) Taiwan
2) Antarctica		     10) China			  18) India		       26) Korea (North)	    34) Mongolia		 42) Réunion		      50) Tajikistan
3) Armenia		     11) Christmas Island	  19) Indonesia		       27) Korea (South)	    35) Myanmar (Burma)		 43) Russia		      51) Thailand
4) Azerbaijan		     12) Cocos (Keeling) Islands  20) Iran		       28) Kuwait		    36) Nepal			 44) Saudi Arabia	      52) Turkmenistan
5) Bahrain		     13) Cyprus			  21) Iraq		       29) Kyrgyzstan		    37) Oman			 45) Seychelles		      53) United Arab Emirates
6) Bangladesh		     14) East Timor		  22) Israel		       30) Laos			    38) Pakistan		 46) Singapore		      54) Uzbekistan
7) Bhutan		     15) French S. Terr.	  23) Japan		       31) Lebanon		    39) Palestine		 47) Sri Lanka		      55) Vietnam
8) Brunei		     16) Georgia		  24) Jordan		       32) Macau		    40) Philippines		 48) Syria		      56) Yemen
#? 10
Please select one of the following timezones.
1) Beijing Time
2) Xinjiang Time, Vostok
#? 1

The following information has been given:

	China
	Beijing Time

Therefore TZ='Asia/Shanghai' will be used.
Selected time is now:	Wed Jul 12 16:18:37 CST 2023.
Universal Time is now:	Wed Jul 12 08:18:37 UTC 2023.
Is the above information OK?
1) Yes
2) No
#? 1

You can make this change permanent for yourself by appending the line
	TZ='Asia/Shanghai'; export TZ
to the file '.profile' in your home directory; then log out and log in again.

Here is that TZ value again, this time on standard output so that you
can use the /usr/bin/tzselect command in shell scripts:
Asia/Shanghai
root@hgbio-server:~# cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime 
root@hgbio-server:~# date
Wed Jul 12 04:27:33 PM CST 2023
```

在联网情况下，会自动更新系统时间。否则可以手动设置时间。`date -s`命令。

2. 使用以下命令重新配置时区：

```shell
sudo dpkg-reconfigure -f noninteractive tzdata
```

3. 把当前准确系统时间写入硬件电子钟。

```shell
sudo hwclock -w
```

4. 格式化查看当前系统时间

```shell
root@hgbio-server:~# date +"%Y-%m-%d %H:%M:%S"
2023-07-12 16:29:57
```

