---
title: maven配置文件加密
date: 2019-02-16 10:16:30
categories: maven
tags:
---

在使用maven私服的时候，我们需要在maven配置文件settings.xml中配置私服的账号密码，但是明文的密码会暴露出去，每个开发人员都能看到，这就有安全风险。很幸运，maven提供了加密这些密码的方式，下面我们就来学习……

参考：http://maven.apache.org/guides/mini/guide-encryption.html

## 获取master密码

改密码是一个用来加密其它密码的密码。  
例如下面，对xrlj.123456加密，作为master的密码。

    D:\apache-maven-3.6.0\conf>mvn --encrypt-master-password xrlj.123456
    {Naja8OoiZ1EHpj5eAL2pD3KVDC/qh0EBXZuvivt53+w=}
    
然后编辑文件`~/.m2/settings-security.xml`这个文件,如果没有，则新建一个。该文件必须在这个目录下。 

编辑添加内容：

    <settingsSecurity>
        <master>{Naja8OoiZ1EHpj5eAL2pD3KVDC/qh0EBXZuvivt53+w=}</master>
    </settingsSecurity>
    
也可以把`settings-security.xml`文件放到指定的目录下，编辑内容和上面一样。但是，但是依然要在`~/.m2`目录中编辑`settings-security.xml` ,指向实际的`settings-security.xml`。如下：

      <settingsSecurity>
          <master>/path/setting-security.xml</master>
      </settingsSecurity>
      
## 加密server的密码

例如，对下面的server加密：

    <server>
        <id>repository</id>
        <username>admin</username>
        <password>admin123</password>
    </server>
    
执行下面命令：

    D:\apache-maven-3.6.0\conf>mvn --encrypt-password admin123
    {K7e0zxQTHJ4HCDH8Wtxj7Lvv0sV2F1qTpAewNVEz7FI=}
    
所以配置改为：

    <server>
        <id>repository</id>
        <username>admin</username>
        <password>{K7e0zxQTHJ4HCDH8Wtxj7Lvv0sV2F1qTpAewNVEz7FI=}</password>
    </server> 
    
--------------- 完  ------------------------           
    