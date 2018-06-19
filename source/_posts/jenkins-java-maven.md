---
title: jenkins-java-maven持续集成（CentOs7）
date: 2017-11-28 09:15:26
categories: 持续集成
tags: jenkins-java-maven
---

传统的开发是开发，打包测试，开发，打包测试，且每次都是全量打包。这给运维，测试带来大量没必要的工作量，同时每次全量打包导致系统每次测试不全面，bug不断，测试，开发，运维叫苦连天。因此持续集成开发势在必行。

参考：  
http://www.jianshu.com/p/a7d7df97fe4b

## 1. 安装并运行jenkins-war
- [下载](https://jenkins.io/download/)
- 运行 `java -jar jenkins.war --httpPort=8080.`
- 在浏览器访问 http://localhost:8080. 记得开防火墙。

图一：
{% asset_img a.png %}

## 2. 初次访问配置
- 按图一红色提示，在服务器对应目录下找到安全密码，拷贝进去登录。
- 按照页面想到，安装插件，如果不确定要安装什么插件，那就选择推荐的插件按钮即可。
- 创建管理员账号，按照页面设置即可。

这样，下面就可以开始使用jenkins了。


## 3. maven，svn,springboot下持续构建java应用
- 安装maven环境。[下载maven](http://mirror.bit.edu.cn/apache/maven/maven-3/3.5.2/binaries/apache-maven-3.5.2-bin.tar.gz)

### 3.1 配置jenkins

{% asset_img b.png %}

点击`系统管理`。

{% asset_img c.png %}

配置各种环境：

- maven setting.xml

{% asset_img e.png %}

- jdk,取消自动安装

{% asset_img f.png %}

- maven,取消自动安装

{% asset_img g.png %}

### 3.2 安装maven插件

{% asset_img h.png %}

-------

{% asset_img i.png %}

点击直接安装即可。

### 3.3 创建新项目

{% asset_img 1.png %}

选定。

{% asset_img 2.png %}

点击ok。

### 3.4 配置项目各种信息
{% asset_img 3.png %}

------------
{% asset_img 4.png %}

### 3.5 开始构建项目

{% asset_img 5.png %}

查看构建结果

{% asset_img 6.png %}

然后再服务器相关目录下就能看到构建后的jar包:
{% asset_img 7.png %}

### 3.6 构建后配置
构建成功后，我们需要发布项目到远程服务器，或者执行等一系列动作。

#### 3.6.1 直接执行jar
修改项目配置：
{% asset_img 8.png %}

添加成功构建后要执行的脚本：
{% asset_img 8_1.png %}

> 注意脚本写法，如果是后台执行：
    `BUILD_ID=dontKillMe nohup /usr/local/SBA/startup.sh &`

#### 3.6.2 发布jar到远程服务器，并执行
 - 下载相关插件：
 {% asset_img 10.png %}
 
- 在系统设置中添加远程服务器：
 {% asset_img 11.png %}

- 在项目中设置，选定远程服务器：
{% asset_img 12.png %}

*注意：*
{% asset_img 13.png %}

- 备份，执行远程服务器上的jar包

1. 在远程服务器上设置脚本，放在目录：`server/bin`：

>停止应用脚本`stop.sh`：
    # 将应用停止
    #stop.sh
    #!/bin/bash
    echo "Stopping SpringBoot Application"
    pid=`ps -ef | grep jenkins-demo*.jar | grep -v grep | awk '{print $2}'`
    if [ -n "$pid" ]
    then
       kill -9 $pid
    fi
---    
> 备份应用脚本`replace.sh`:
    #replace.sh 用于将上次构建的结果备份，然后将新的构建结果移动到合适的位置
    #!/bin/bash
    # 先判断文件是否存在，如果存在，则备份
    file="/server/java/apps/jenkins-demo.jar"
    if [ -f "$file" ]
    then
       mv /server/java/apps/jenkins-demo.jar /server/java/apps/backup/jenkins-demo.jar.`date +%Y%m%d%H%M%S`
    fi
    mv /server/java/apps/jenkins-demo*.jar /server/java/apps/jenkins-demo.jar
---
> 启动应用包脚本`start.sh`:
    # startup.sh 启动项目
    #!/bin/bash
    echo "授予当前用户权限"
    chmod 777 /server/java/apps/jenkins-demo.jar
    echo "执行....."
    java -jar /server/java/apps/jenkins-demo.jar
   注意：启动脚本如果是放在和jenkins同一个服务器上，后台执行`nohub java -jar /server/java/apps/jenkins-demo.jar &`不起作用，所在要在jenkins上这样配置`BUILD_ID=dontKillMe nohup /usr/local/SBA/startup.sh &`  
   如果是远程服务器，就可以改成：
       # startup.sh 启动项目
       #!/bin/bash
       echo "授予当前用户权限"
       chmod 777 /server/java/apps/jenkins-demo.jar
       echo "执行....."
       nohup java -jar /server/java/apps/jenkins-demo.jar > jenkins-demo.out 2>& 1 &
    
---
2. 设置远程服务器执行脚本： 

{% asset_img 14.png %}   

如果是jenkins所在服务器执行后台jar，要注意，远程服务器其实可以去掉，在脚本改成后台执行就可以。

{% asset_img 15.png %}   

## 备注：

1.替换jar包中的文件

`jar -uvf rpds.jar BOOT-INF/classes/db.properties `

2.启动jar应用程序shell脚本

    #!/bin/sh
    # chkconfig: 2345 64 36
    ## java env 
    #################### zhengjianyan #######################
    export JAVA_HOME=/server/java/jdk
    export JRE_HOME=$JAVA_HOME/jre
    
    BASE_PATH=/server/java/service/com.xcsqjr.scf.service
    VERSION=1.0.0
    ## service name
    APP_NAME=all
    
    SERVICE_DIR=$BASE_PATH/com.xcsqjr.scf.service.$APP_NAME
    SERVICE_NAME=com.xcsqjr.scf.service.$APP_NAME-$VERSION
    #JAR_NAME=$SERVICE_DIR/$SERVICE_NAME/target\.jar
    JAR_NAME=$SERVICE_DIR/target/$SERVICE_NAME.jar
    PID=$SERVICE_NAME\.pid
    
    rm -rf $SERVICE_DIR/target/*.tmp
    cd $BASE_PATH
    cd ..
    
    $JAVA_HOME/bin/jar uvf $SERVICE_DIR/target/$SERVICE_NAME.jar application.properties
    
    cd $SERVICE_DIR
    case "$1" in
    
        start)
            nohup $JRE_HOME/bin/java -Xms128m -Xmx256m -jar $JAR_NAME >/dev/null 2>&1 &
            echo $! > $SERVICE_DIR/target/$PID
            echo "=== start $SERVICE_NAME"
            ;;
    
        stop)
            kill `cat $SERVICE_DIR/target/$PID`
            rm -rf $SERVICE_DIR/target/$PID
            echo "=== stop $SERVICE_NAME"
    
            sleep 5
            P_ID=`ps -ef | grep -w "$SERVICE_NAME" | grep -v "grep" | awk '{print $2}'`
            if [ "$P_ID" == "" ]; then
                echo "=== $SERVICE_NAME process not exists or stop success"
            else
                echo "=== $SERVICE_NAME process pid is:$P_ID"
                echo "=== begin kill $SERVICE_NAME process, pid is:$P_ID"
                kill -9 $P_ID
            fi
            ;;
    
        restart)
            $0 stop
            sleep 2
            $0 start
            echo "=== restart $SERVICE_NAME"
            ;;
    
        *)
    		echo "==== restart ====="
            ## restart
            $0 stop
            sleep 5
            $0 start
            ;;
    esac
    exit 0



