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
    
    或者(如果么有mvnw)
    
    mvn install dockerfile:build
    
        
## 上传到docker私服

1.配置setting.xml

    <server>
      <id>ip:8082</id>
      <username>admin</username>
      <password>admin123</password>
    </server>
    
注意id，必须为私服地址。


2.配置pom.xml

    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
        <!-- tag::plugin[] -->
        <plugin>
            <groupId>com.spotify</groupId>
            <artifactId>dockerfile-maven-plugin</artifactId>
            <version>1.4.9</version>
            <configuration>
                <repository>119.145.41.171:8082/${docker.image.prefix}/${project.artifactId}</repository>
                <tag>${project.version}</tag>
                <useMavenSettingsForAuth>true</useMavenSettingsForAuth>
            </configuration>
        </plugin>
        <!-- end::plugin[] -->

        <!-- tag::unpack[] -->
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
        <!-- end::unpack[] -->
    </plugins>
    
重点两个参数： 

- repository：需要加上私服地址。
- useMavenSettingsForAuth：设置为true。

3.执行命令构建并上传镜像到私服

    mvn clean install dockerfile:push   
    
## 绑定maven执行阶段

    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
        <!-- tag::plugin[] -->
        <plugin>
            <groupId>com.spotify</groupId>
            <artifactId>dockerfile-maven-plugin</artifactId>
            <version>1.4.9</version>
            <executions>
                <execution>
                  <id>default</id>
                  <goals>
                    <goal>build</goal>
                    <goal>push</goal>
                  </goals>
                </execution>
             </executions>
            <configuration>
                <repository>119.145.41.171:8082/${docker.image.prefix}/${project.artifactId}</repository>
                <tag>${project.version}</tag>
                <useMavenSettingsForAuth>true</useMavenSettingsForAuth>
            </configuration>
        </plugin>
        <!-- end::plugin[] -->

        <!-- tag::unpack[] -->
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
        <!-- end::unpack[] -->
    </plugins>

这样子就可以使用`mvn package`,`mvn deploy`命令了。也可以精准使用`mvn dockerfile:build`。          
    
## 启动容器

    $ docker run -e "SPRING_PROFILES_ACTIVE=prod" -p 8080:8080 -t springio/gs-spring-boot-docker
   
或者

    $ docker run -e "SPRING_PROFILES_ACTIVE=dev" -p 8080:8080 -t springio/gs-spring-boot-docker
    

## 调试容器内的应用

                 