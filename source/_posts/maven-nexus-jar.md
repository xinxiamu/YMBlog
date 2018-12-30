---
title: maven构建jar上传到nexus私服并拉取
date: 2018-12-30 14:57:05
categories: maven
tags:
---

公司内部有很多自己的java lib库等共用代码，为了在公司内部自由共享这些资源，所以需要搭建maven私服。下面介绍利用maven构建java项目为jar包并上传到nexus私服，以及在其它项目中如何引用私服中的jar包。


## nexus3的安装

这里不做介绍，参考章节{%post_link centos-maven-nexus%},或者{%post_link docker-app-install%}。 

## 新建maven2仓库

{%asset_img a-1.png%}

说明：

- 仓库名称：ymu-hosted，类型为hosted。maven仓库有三种类型，hosted、proxy、group。hosted代表私服本机服务器，proxy代理外部仓库服务器，group整合各个hosted和proxy，按顺序策略获取jar。
- 这里选择Version policy为：Release。正式发布版本。正式发布版本都上传到这里来。
- Deployment policy: 选择为Allow Redeploy，可重复发布。代表同一个版本号的jar可以多次发布。这样就不必要设置快照仓库了，项目小改动也作为正式版本发布上去，不必要修改版本号。


## maven构建jar并上传到私服相应仓库

### 配置pom.xml

    <!--mavne 发布-->
    <distributionManagement>
        <snapshotRepository>
            <id>snapshots</id>
            <name>maven-snapshots</name>
            <url>http://ip:8085/repository/maven-snapshots/</url>
        </snapshotRepository>
        <repository>
            <id>repository</id>
            <name>ymu-hosted</name>
            <url>http://ip:8085/repository/ymu-hosted/</url>
        </repository>
    </distributionManagement>

说明： 
-  snapshotRepository：指定jar包要发布的快照仓库。版本号这样`0.0.1-SNAPSHOT`，代表的是快照版本，后缀的`SNAPSHOT`必须是大写，不能是小写。
- repository：指定jar包发布上去的正式版本仓库。当版本号是这样`<version>0.0.2</version>`，没有`SNAPSHOT`的时候，jar将发布到这里指定的私服仓库ymu-hosted下。
- id：对应maven配置setting.xml中的配置。保持id一致。远程仓库的唯一标志，很重要。
- name：只是方便阅读，随意定义。   
- url：仓库地址。登录nexus，在配置中可以查看。

### 配置setting.xml

    <server>
      <id>repository</id>
      <username>admin</username>
      <password>admin123</password>
    </server>

    <server>
      <id>snapshots</id>
      <username>admin</username>
      <password>admin123</password>
    </server>

配置完成后，执行`mvn deploy`命令，就可以发布jar到私服了。修改版本号，可以实际验证查看发布到哪个仓库下。 


## 从nexus私服下载构件

配置pom.xml:

    <repositories>
        <!--自己私仓,下载jar-->
        <repository>
            <id>ymu-public</id>
            <name>ymu nexus</name>
            <url>http://119.145.41.171:8085/repository/maven-public/</url>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
            <releases>
                <enabled>true</enabled>
            </releases>
        </repository>

        
    </repositories>

    <pluginRepositories>
        <!--私服，插件-->
        <pluginRepository>
            <id>ymu-public</id>
            <name>ymu nexus</name>
            <url>http://119.145.41.171:8085/repository/maven-public/</url>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </pluginRepository>
        
    </pluginRepositories>

看上面配置，我们配置自己的私服用的是`maven-public`仓库，该仓库是group类型，整合了自己的私仓等，如下图：

{%asset_img a-2.png%}

看Group组，我们可以看到Member里面有很多个仓库，这代表，拉取jar构件的时候，首先从私服本机拉取，拉取不到的话再从下一个拉取，按顺序处理。从maven中央仓库拉取的jar构件都会缓存到其它仓库。  

以上，我们看到的配置在pom.xml中，只针对某个项目有效。如果是多个项目想都通用私服，则可以在maven的setting.xml中配置，这里不做介绍。个人更喜欢在pom.xml配置，放到父pom中。

## 上传第三个jar包到nexus私服中

{%asset_img a-3.png%}

如上图，点击`maven-releases`仓库，进入下面图：

{%asset_img a-4.png%}

按照输入要求，下面我们上传一个第三方的jar包：

{%asset_img a-5.png%}

最好选定下生成pom.xml，否则在其它项目引用的时候，无法点击跳进去。但是对jar使用没影响。

可以打开对应仓库浏览已上传的jar构件：

{%asset_img a-6.png%}  

下面就可以在pom.xml中正常的引用了。