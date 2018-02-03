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
