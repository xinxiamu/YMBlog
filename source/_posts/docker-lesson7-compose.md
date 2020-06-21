---
title: docker学习-第七课：三剑客之Docker Compose
date: 2018-08-01 20:33:32
categories: docker
tags:
---

在一个例如web应用中，除了应用本身之外，其还关联用到数据库等……，如果都采用docker化的话，那么就需要启动所有相关的docker容器。那么，有没有可能，按照某种顺序，对所有相关联的docker容器进行编排，然后按照一定的先后顺序依次启动。是的，Docker Compose就是这么一个工具，容器编排工具。定义和运行多个容器的应用。 

Compose中有两个重要的概念：

- 服务（Service）：一个应用的容器,实际上可以包括若干运行相同镜像的容器实例。
- 项目(project)：由一组关联的应用容器组成的一个完整业务单元,在	docker-compose.yml文件中定义。

Compose	的默认管理对象是项目,通过子命令对项目中的一组容器进行便捷地生命周期管理。

## 安装Docker Compose

- 确保已经安装了docker引擎，docker compose依赖docker引擎。
- docker compose不要在root账户权限下运行。（为啥？在root运行下会有什么问题吗，是否也可以？）

Docker Compose安装：https://docs.docker.com/compose/install/

### CentOs7下安装

_注意：_ For alpine, the following dependency packages are needed: py-pip, python-dev, libffi-dev, openssl-dev, gcc, libc-dev, and make.

1.执行下面命令获取合适版本：

    sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

安装不同版本，只需要更改`1.24.1`。
   
2.给二进制可执行文件赋权：

    sudo chmod +x /usr/local/bin/docker-compose
    
这样，Docker Compose就安装完成了。

3.测试是否安装成功：

    [root@izwz9guplfsq8ltfk0wnjiz ~]# docker-compose --version
    docker-compose version 1.24.1, build 4667896b
    
看上面输出信息可知已经安装成功了。

## 安装Docker Compose命令补全工具

我们已经安装了Compose，但是在命令窗口输入docker-compose后，按下Tab键，没有显示其所有具有的命令。因此，我们需要安装命令补全工具。

命令补全工具安装：https://docs.docker.com/compose/completion/

安装命令补全工具只需要执行下面命令：

    curl -L https://raw.githubusercontent.com/docker/compose/$(docker-compose version --short)/contrib/completion/bash/docker-compose -o /etc/bash_completion.d/docker-compose

这样，在重新登录后，输入docker-compose 并按下Tab键，Compose就可自动补全命令了。

_注:_网址`raw.githubusercontent.com`可能需要翻墙才能访问。或者配置`hosts`,添加内容：`199.232.28.133 raw.githubusercontent.com`,亲测有效。

## 快速入门

这里展示一个简单的示例，简单了解docker compose使用步骤。

### 基本步骤：

1.使用Dockerfile（或其他方式）定义应用程序环境，以便在任何地方重现该环境。     
2.在docker-compose.yml文件中定义组成应用程序的服务，以便各个服务在一个隔离的环境中一起运行。       
3.运行docker-compose up命令，启动并运行整个应用程序。

### 入门示例

- 新建文件夹`docker-compose-demo`,进入该目录，拷贝文件`eureka-server.jar`进来。

- 新建文件`Dockerfile`，添加内容：


    FROM java:8
    VOLUME /tmp
    ADD eureka-server.jar app.jar
    RUN bash -c 'touch /app.jar'
    EXPOSE 9000
    ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
    
- 再新建文件`docker-compose.yml`,添加内容：


    version: '2'			# 表示该docker-compose.yml文件使用的是Version 2 file format
    services:
      eureka:				# 指定服务名称
        build: .			# 指定Dockerfile所在路径
        ports:
          - "8761:8761"		# 指定端口映射，类似docker run的-p选项，注意使用字符串形式           
          
- 在docker-compose.yml 所在路径执行以下命令。     
  

    docker-compose up
    
Compose就会自动构建镜像并使用镜像启动容器。我们也可使用docker-compose up -d 后台启动并运行这些容器。     

- 访问：http://宿主机IP:8761/ ，即可访问Eureka Server首页   

### 工程、服务、容器

Docker Compose将所管理的容器分为三层，分别是工程（project），服务（service）以及容器（container）。Docker Compose运行目录下的所有文件（docker-compose.yml, extends文件或环境变量文件等）组成一个工程（默认为docker-compose.yml所在目录的目录名称）。一个工程可包含多个服务；每个服务中定义了容器运行的镜像、参数和依赖，一个服务可包括多个容器实例。

对应《入门示例》一节，工程名称是docker-compose.yml所在的目录名。该工程包含了1个服务，服务名称是eureka；执行docker-compose up时，启动了eureka服务的1个容器实例。

        