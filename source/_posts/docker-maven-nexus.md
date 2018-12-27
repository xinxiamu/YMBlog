---
title: 使用maven插件构建docker镜像并注册到nexus私仓
date: 2018-12-27 14:29:17
categories: docker
tags:
---

Maven是一个强大的项目管理和构建工具，下面我们介绍利用Maven插件构建Docker镜像并注册到Nexus私仓里面，这样，在服务器或者其它地方就可以直接拉取镜像并运行，这对服务共享或者服务器部署应用起到极大的方便。      
在maven中央仓库，可以搜索到好几个docker-maven-pluging。

下面我们采用一款由Spotify公司开发的Maven插件[docker-maven-plugin](https://github.com/spotify/docker-maven-plugin)。

参考网址：   
http://itmuch.com/docker/12-docker-maven/   
https://blog.csdn.net/aixiaoyang168/article/details/77453974




        <plugin>
            <groupId>com.spotify</groupId>
            <artifactId>docker-maven-plugin</artifactId>
            <version>0.4.13</version>
            <configuration>
                <imageName>http://119.145.41.171:8082/mavendemo:v1.0</imageName>
                <baseImage>java</baseImage>
                <maintainer>docker_maven docker_maven@email.com</maintainer>
                <workdir>/ROOT</workdir>
                <cmd>["java", "-version"]</cmd>
                <entryPoint>["java", "-jar", "${project.build.finalName}.jar"]</entryPoint>
                <resources>
                    <resource>
                        <targetPath>/ROOT</targetPath>
                        <directory>${project.build.directory}</directory>
                        <include>${project.build.finalName}.jar</include>
                    </resource>
                </resources>
                <serverId>my-docker-registry</serverId>
            </configuration>
            <executions>
                <execution>
                    <id>build-image</id>
                    <phase>package</phase>
                    <goals>
                        <goal>build</goal>
                    </goals>
                </execution>
                <execution>
                    <id>tag-image</id>
                    <phase>package</phase>
                    <goals>
                        <goal>tag</goal>
                    </goals>
                    <configuration>
                        <image>mavendemo:${project.version}</image>
                        <newName>http://119.145.41.171:8082/mavendemo:v1.0</newName>
                    </configuration>
                </execution>
                <execution>
                    <id>push-image</id>
                    <phase>deploy</phase>
                    <goals>
                        <goal>push</goal>
                    </goals>
                    <configuration>
                        <imageName>http://119.145.41.171:8082/mavendemo:v.10</imageName>
                    </configuration>
                </execution>
            </executions>
        </plugin>