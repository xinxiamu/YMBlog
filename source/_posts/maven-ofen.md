---
title: maven使用经验记录
date: 2018-02-03 10:00:23
categories: maven
tags: maven常用功能
---

本文记录在开发过程中经常性使用到的maven特性……

### 导出maven依赖的jar包到目录

方法一：在pom目录下执行命令	
`mvn dependency:copy-dependencies`

方法二：eclipse项目下：
选择项目的pom.xml文件，点击右键菜单中的Run As,见下图红框中，在弹出的Configuration窗口中，输入 dependency:copy-dependencies后，点击运行
maven项目所依赖的jar包会导出到targed/dependency目录中。

{% asset_img a.jpg %} 

{% asset_img b.jpg %} 

### maven打包时候跳过test检查

方法一： 在pom目录下执行命令
`mvn clean install -Dmaven.test.skip=true`

方法二： 在pom中添加插件

    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <configuration>
            <skipTests>true</skipTests>
        </configuration>
    </plugin>
    
### 指定测试或者开发等环境执行命令

```shell script
mvn clean install deploy -Pprod
```   
关键是`-Pprod`这里，这里表示指定prod环境执行。 

### 指定settings.xml执行

```shell script
mvn install --settings E:\apache-maven-3.3.9\conf\settings-ztesoft.xml -Dmaven.test.skip=true
```

### 查看环境

```text
$ mvn help:active-profiles 列出当前激活的Profile
$ mvn help:all-profiles 列出当前所有的Profile
```

### 打包jar

#### 普通打包
会把maven依赖包一起打包。

    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <configuration>
            <source>1.8</source>
            <target>1.8</target>
        </configuration>
    </plugin>

#### 不打包lib
在spring boot中实践，不打包maven依赖包，减少打包后jar的大小。 
让后启动指定main类，并指定依赖包目录lib。

    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-jar-plugin</artifactId>
        <configuration>
            <archive>
                <manifest>
                    <mainClass>com.xcsqjr.StartAbsServiceAllApplication</mainClass>
                    <addClasspath>true</addClasspath>
                    <classpathPrefix>lib/</classpathPrefix>
                </manifest>
                <manifestEntries>
                    <Class-Path>./</Class-Path>
                </manifestEntries>
            </archive>
        </configuration>
    </plugin>


## jar包无法更新下来，爆本地已有缓存

有时候，maven依赖包拉取失败，但是有部分内容已经拉取，一部分没拉去，再次更新的时候，不会再次去远程仓库拉取。所以会报错误：

    Maven-010-maven 编译报错：Failure to ... in ... was cached in the local repository, resolution will not be reattempted until the update interval of nexus has elapsed or updates are forced.


解决办法是，在maven配置文件setting.xml中加入如下策略：

    <profile>
      <id>dev</id>
        <repositories>
            <!--自己私仓,下载jar-->
            <repository>
                <id>ymu-public</id>
                <name>ymu nexus</name>
                <url>http://119.145.41.171:8085/repository/maven-public/</url>
                <snapshots>
                    <enabled>true</enabled>
                    <updatePolicy>always</updatePolicy>
                </snapshots>
                <releases>
                    <enabled>true</enabled>
                    <updatePolicy>always</updatePolicy>
                </releases>
            </repository>

            <repository>
                <id>central</id>
                <url>http://central</url>
                <releases>
                    <enabled>true</enabled>
                    <updatePolicy>always</updatePolicy>
                </releases>
                <snapshots>
                    <enabled>true</enabled>
                    <updatePolicy>always</updatePolicy>
                </snapshots>
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
                    <updatePolicy>always</updatePolicy>
                </snapshots>
                <releases>
                    <enabled>true</enabled>
                    <updatePolicy>always</updatePolicy>
                </releases>
            </pluginRepository>
            <pluginRepository>
                <id>central</id>
                <url>http://central</url>
                <releases>
                    <enabled>true</enabled>
                    <updatePolicy>always</updatePolicy>
                </releases>
                <snapshots>
                    <enabled>true</enabled>
                    <updatePolicy>always</updatePolicy>
                </snapshots>
            </pluginRepository>
        </pluginRepositories>
    </profile>
    
关键加上属性`<updatePolicy>always</updatePolicy> `

## 常用maven中央仓库地址

    1、http://www.sonatype.org/nexus/  私服nexus工具使用
    2、http://mvnrepository.com/ （推荐）
    3、http://repo1.maven.org/maven2
    
    4、http://maven.aliyun.com/nexus/content/groups/public/  阿里云  （强力推荐）
    
    5、http://repo2.maven.org/maven2/ 私服nexus工具使用
    
    6、http://uk.maven.org/maven2/
    
    7、http://repository.jboss.org/nexus/content/groups/public
    
    8、http://maven.oschina.net/content/groups/public/  oschina可惜啦，以前一直用这个，不过现在有阿里云来擦屁股啦
    
    9、http://mirrors.ibiblio.org/maven2/
    
    10、http://maven.antelink.com/content/repositories/central/
    
    11、http://nexus.openkoala.org/nexus/content/groups/Koala-release/
    
    12、http://maven.tmatesoft.com/content/groups/public/ 
    
    ---------------------- spring的 -------------------------
    
    <repository>
        <id>spring-snapshots</id>
        <name>Spring Snapshots</name>
        <url>https://repo.spring.io/libs-snapshot-local</url>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
    </repository>
    <repository>
        <id>spring-milestones</id>
        <name>Spring Milestones</name>
        <url>https://repo.spring.io/libs-milestone-local</url>
        <snapshots>
            <enabled>false</enabled>
        </snapshots>
    </repository>
    <repository>
        <id>spring-releases</id>
        <name>Spring Releases</name>
        <url>https://repo.spring.io/libs-release-local</url>
        <snapshots>
            <enabled>false</enabled>
        </snapshots>
    </repository>  
    
    
## 从私服无法拉取构件

- 方法一：  
上传第三方jar到私服，再应用。

- 方法二：  
找到第三方jar包所在的第三方私服，然后在自己的私服中做代理请求。    

{% asset_img a-1.png %} 

可以看到里面代理了很多第三方私服，阿里云的，spring的，等。然后整合到`maven-pblic`group组中即可。

## maven配置修改，指定java环境

```shell
[root@iZ7xvc7no7w1i3tcdv8wkgZ bin]# pwd
/server/java/maven/bin
[root@iZ7xvc7no7w1i3tcdv8wkgZ bin]# ls
m2.conf  mvn  mvn.cmd  mvnDebug  mvnDebug.cmd  mvnyjp
[root@iZ7xvc7no7w1i3tcdv8wkgZ bin]# 

```

编辑`mvn`文件：

```shell
# -----------------------------------------------------------------------------
# Apache Maven Startup Script
#
# Environment Variable Prerequisites
#
#   JAVA_HOME       Must point at your Java Development Kit installation.
#   MAVEN_OPTS      (Optional) Java runtime options used when Maven is executed.
#   MAVEN_SKIP_RC   (Optional) Flag to disable loading of mavenrc files.
# -----------------------------------------------------------------------------
JAVA_HOME=/server/java/jdk-11.0.7
```

`JAVA_HOME`配置java环境。
