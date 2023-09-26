---
title: Ubuntu服务器运维
date: 2023-06-07 15:01:24
categories: ubuntu
tags: ubuntu server
---

## 局域网下设置静态IP

要在Ubuntu Server中为网络接口`eno1`和`eno2`设置静态IP地址，可以修改网络配置文件。以下是操作步骤：

1. 打开终端，运行以下命令以编辑网络配置文件：
   ```bash
   sudo nano /etc/netplan/01-netcfg.yaml
   ```

2. 修改文件以为接口设置静态IP地址。用以下内容替换现有的配置：
   ```yaml
   network:
     version: 2
     renderer: networkd
     ethernets:
       eno1:
         dhcp4: no
         addresses: [192.168.0.100/24]
         nameservers:
            addresses: [8.8.8.8, 8.8.4.4]  # 根据你的 DNS 服务器进行修改
         routes:
          - to: 0.0.0.0/0
            via: 192.168.0.1
   ```
   在上述配置中，`eno1`被分配IP地址`192.168.0.100`，子网掩码为`/24`（等同于255.255.255.0）。

3. 保存更改并退出编辑器，按下Ctrl + X，然后输入Y，最后按下Enter键。

4. 应用网络配置更改，运行以下命令：
   ```bash
   sudo netplan apply
   ```

5. 重启网络服务以使更改生效：
   ```bash
   sudo systemctl restart systemd-networkd
   ```

按照上述步骤后，网络接口`eno1`应具有您指定的静态IP地址。可以在终端中运行命令`ip addr show eno1`来验证配置。

## 挂载硬盘

