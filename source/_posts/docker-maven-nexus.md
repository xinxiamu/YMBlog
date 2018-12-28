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

## 快速入门

1.新建spring-cloud项目，eureka-server。

2.配置pom.xml文件：
    
    <plugin>
        <groupId>com.spotify</groupId>
        <artifactId>docker-maven-plugin</artifactId>
        <version>1.2.0</version>
        <configuration>
            <imageName>my-image:${project.version}</imageName>
            <baseImage>java</baseImage>
            <entryPoint>["java", "-jar", "/${project.build.finalName}.jar"]</entryPoint>
            <!-- copy the service's jar file from target into the root directory of the image -->
            <resources>
                <resource>
                    <targetPath>/</targetPath>
                    <directory>${project.build.directory}</directory>
                    <include>${project.build.finalName}.jar</include>
                </resource>
            </resources>
        </configuration>
    </plugin> 

插件配置说明：

- imageName：指定镜像名称。一般是用`仓库名称+实际名字`作为镜像名称，如：`docker-repo/my-image`整个作为镜像名词，也可以直接用`my-image`作为镜像名称。冒号后面是镜像tag，如果不指定，则tag是`latest`,所以一般指定，就直接用maven项目的版本号。这里特别注意，tab不能带大写字母，否则报错。比如maven的版本号默认：0.0.1-SNAPSHOT。这样构建将会报错误。去掉`SNAPSHOT`。
- baseImage：用于指定基础镜像，类似于Dockerfile中的FROM指令。这里用了`java`基础镜像，这是个集成了openJdk1.8的镜像，有643M那么大。因此，我们可以用这个替换[hub.docker.com](https://hub.docker.com/r/anapsix/alpine-java),只有127M大小，集成的jdk8。
- entrypoint：类似于Dockerfile的ENTRYPOINT指令。
- resources.resource.directory：用于指定需要复制的根目录，${project.build.directory}表示target目录。会在target目录下生成docker目录，jar包等生成在里面。
- resources.resource.include：指定需要复制的文件。${project.build.finalName}.jar指的是打包后的jar包文件。

3.设置环境变量DOCKER_HOS

编辑/etc/profile文件，增加环境变量：
    
    export DOCKER_HOST=tcp://192.168.33.10:2375

ip是指本机ip。端口指定，不可更改。

4.启动docker引擎，执行下面命令开始构建：

    mvn clean package docker:build
    
    --查看是否生成镜像
    docker image ls

5.启动镜像：

     docker run --name eureka-server -p 8084:8084 -d my-image

可以正常访问网站啦！！      
    
## 使用Dockerfile进行构建

    <build>
      <plugins>
        ...
        <plugin>
          <groupId>com.spotify</groupId>
          <artifactId>docker-maven-plugin</artifactId>
          <version>VERSION GOES HERE</version>
          <configuration>
            <imageName>example</imageName>
            <dockerDirectory>docker</dockerDirectory>
            <resources>
               <resource>
                 <targetPath>/</targetPath>
                 <directory>${project.build.directory}</directory>
                 <include>${project.build.finalName}.jar</include>
               </resource>
            </resources>
          </configuration>
        </plugin>
        ...
      </plugins>
    </build>

 基本不变，改成这个属性`<dockerDirectory>docker</dockerDirectory>` 。指定Dockerfile所在目录即可。然后也是执行`mvn clean package docker:build`就可构建了。     

## 构建docker镜像并push到docker私服




        <plugin>
            <groupId>com.spotify</groupId>
            <artifactId>docker-maven-plugin</artifactId>
            <version>0.4.13</version>
            <configuration>
                <imageName>http://119.145.41.171:8082/mavendemo:v1.0</imageName>
                <baseImage>java</baseImage>
                <dockerDirectory>${project.basedir}/src/main/docker</dockerDirectory>
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