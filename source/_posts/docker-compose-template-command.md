---
title: docker-compose.yml常用命令
date: 2018-08-01 20:33:35
categories: docker
tags:
---

默认的模板文件是 docker-compose.yml ，其中定义的每个服务都必须通过 image 指令指定镜像或
build 指令（需要 Dockerfile）来自动构建。

## image

指定为镜像名称或者id，如果本地没有该镜像，将会到配置的镜像仓库拉取该镜像。

例如：

    image: ubuntu
    image: orchardup/postgresql
    image: a4bc65fd
    
## build

指定`Dockerfile`所在文件夹的路径。Compose将会利用它自动构建这个镜像，然后使用这个镜像。

    build: /path/to/build/dir
    
也可以是一个对象，用于指定Dockerfile和参数，例如：

    build:
      context: ./dir
      dockerfile: Dockerfile-alternate
      args:
        buildno: 1
        
## command

覆盖容器启动后默认执行的命令。

    command: bundle exec thin -p 3000
    
也可以是一个list，类似于Dockerfile中的CMD指令，格式如下：

    command: [bundle, exec, thin, -p, 3000]
    
## links

链接到其它服务中的容器。使用服务名称（同时作为别名）或服务名称：服务别名（SERVICE:ALIAS） 格
式都可以。

    links:
    - db
    - db:database
    - redis
    
使用的别名将会自动在服务容器中的 /etc/hosts 里创建。例如：

    172.17.2.186 db
    172.17.2.186 database
    172.17.2.187 redis
    
相应的环境变量也将被创建。

## external_links

链接到`docker-compose.yml`外部的容器，甚至并非Compose管理的容器。参数格式跟links类似。  

    external_links:
     - redis_1
     - project_db_1:mysql
     - project_db_1:postgresql
     
## ports

暴露端口信息。

使用宿主：容器（HOST:CONTAINER）格式或者仅仅指定容器的端口（宿主将会随机选择端口）都可以。  

需要注意的是，当使用HOST:CONTAINER 格式映射端口时，容器端口小于60将会得到错误的接口，因为yaml会把xx:yy 的数字解析为60进制。因此，建议使用字符串的形式。示例：

    ports:
     - "3000"
     - "3000-3005"
     - "8000:8000"
     - "9090-9091:8080-8081"
     - "49100:22"
     - "127.0.0.1:8001:8001"
     - "127.0.0.1:5000-5010:5000-5010"

## expose

暴露端口，但不映射到宿主机，只被连接的服务访问。

    expose:
     - "3000"
     - "8000"
     
## volumes

卷挂载路径设置。可以设置宿主机路径（HOST:CONTAINER） ，也可指定访问模式（HOST:CONTAINER:ro）。示例：
     
     volumes:
       # Just specify a path and let the Engine create a volume
       - /var/lib/mysql
     
       # Specify an absolute path mapping
       - /opt/data:/var/lib/mysql
     
       # Path on the host, relative to the Compose file
       - ./cache:/tmp/cache
     
       # User-relative path
       - ~/configs:/etc/configs/:ro
     
       # Named volume
       - datavolume:/var/lib/mysql
       
## volumes_from

从另外一个服务或者容器挂载它的所有卷。 可指定只读（ro）或读写（rw），默认是读写（rw）。示例：

    volumes_from:
     - service_name
     - service_name:ro
     - container:container_name
     - container:container_name:rw
      
## environment

设置环境变量，可以使用数组或字典两种格式。

只给定名称的变量会自动获取它在 Compose 主机上的值，可以用来防止泄露不必要的数据。

    environment:
      RACK_ENV: development
      SHOW: 'true'
      SESSION_SECRET:
    
    environment:
      - RACK_ENV=development
      - SHOW=true
      - SESSION_SECRET
      
## env_file

从文件中获取环境变量，可以为单独的文件路径或列表。

如果使用`docker-compose -f FILE`指定模板文件，则`env_file`中路径会基于模板文件路径。

如果有变量和`environment`指令冲突，则以后者为准。即使用environment指定的环境变量会覆盖env_file指定的环境变量

    env_file: .env
    
    env_file:
      - ./common.env   # 共用
      - ./apps/web.env # web用
      - /opt/secrets.env # 密码用            

## extends       

          
