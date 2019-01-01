---
title: spring-boot集成docker部署
date: 2019-01-01 21:13:02
categories: spring-boot
tags:
---

本章介绍spring-boot项目docker化构件，利用maven插件把spring-boot项目构件成docker镜像，并上传到自己的docker私服nexus3上。

官方参考例子： 
https://spring.io/guides/gs/spring-boot-docker/

## 快速在本机构件spring-boot项目镜像

1.新建spring-boot项目。
通过https://start.spring.io/  
2.在项目根目录下添加Dockerfile文件，并编辑内容如下：

    FROM openjdk:8-jdk-alpine
    VOLUME /tmp
    ARG DEPENDENCY=target/dependency
    COPY ${DEPENDENCY}/BOOT-INF/lib /app/lib
    COPY ${DEPENDENCY}/META-INF /app/META-INF
    COPY ${DEPENDENCY}/BOOT-INF/classes /app
    ENTRYPOINT ["java","-cp","app:app/lib/*","hello.Application"]    
3.配置maven插件，编辑pom.xml，添加：
[dockerfile-maven 插件](https://github.com/spotify/dockerfile-maven)

    <properties>
       <docker.image.prefix>springio</docker.image.prefix>
    </properties>
    <build>
        <plugins>
            <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>dockerfile-maven-plugin</artifactId>
                <version>1.4.9</version>
                <configuration>
                    <repository>${docker.image.prefix}/${project.artifactId}</repository>
                    <tag>${project.version}</tag>
                </configuration>
            </plugin>
        </plugins>
    </build>
    
参数说明：
- repository：指定镜像名称。
- tag: 指定镜像标签，如果不指定，则默认是latest。   
    
为了确保在构件docker镜像之前，spring-boot jar包是解压的，添加下面插件配置：

    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-dependency-plugin</artifactId>
        <executions>
            <execution>
                <id>unpack</id>
                <phase>package</phase>
                <goals>
                    <goal>unpack</goal>
                </goals>
                <configuration>
                    <artifactItems>
                        <artifactItem>
                            <groupId>${project.groupId}</groupId>
                            <artifactId>${project.artifactId}</artifactId>
                            <version>${project.version}</version>
                        </artifactItem>
                    </artifactItems>
                </configuration>
            </execution>
        </executions>
    </plugin>
   
4.执行命令构件docker镜像

    $ ./mvnw install dockerfile:build
    
        

    

    
        