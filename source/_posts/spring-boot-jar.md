---
title: spring-boot发布包jar的秘密
date: 2018-10-19 17:27:44
categories: spring-boot
tags: spring-boot-jar
---

### 修改spring-boot可执行jar包

本主题介绍的实际是对java可执行jar包的修改问题。 

在生产环境中，有时候我们发现了个小bug，开发迅速修改代码后，很多时候我们不得不重新发布一个新的可执行jar包上去替换掉。   
但是这样就有个问题了，如果开发人员改动了很多的源码，这样我们就不得不重新测试检查各个功能了。而在生产环境，我们只是想仅仅替换改动的一点点东西。 

在war包运行的情况下，我们可以直接在tomcat对应应用解压文件夹下替换某个文件即可。但是打成jar包就没那么方便了。

所以，这里介绍两种方式达成目的：只换jar包中的某个文件资源：

#### 方法一：用解压工具

1.下载服务器中的jar包。  
2.用360等相关解压工具直接双击`jar`包，打开。  
3.拖动改动后的文件进去覆盖`jar`中的。  
4.关闭解压工具软件。 
5.重新上传改动后的`jar`包到服务器。   
6.执行查看改动后效果。

_注意：_ 整个过程不能解压下载下来的`jar`包。

#### 方法二：java命令

1.下载服务器jar包。    
2.解压jar包

    shell>jar xvf micro-service-core-0608-5-SC-SNAPSHOT.jar

解压后三个目录：
BOOT-INF、META-INF、org

3.把修改过的文件在BOOT-INF下对应的文件夹中覆盖

4.重新打回jar包
     
注意：不能覆盖META-INF下面的MANIFEST.MF文件，不能压缩打包。所以直接用下面命令行打包即可。
     
    shell>jar cvf0M core.jar BOOT-INF META-INF org

执行完在当前目录下应该出现core.jar的新jar包。

5.验证新jar包是否可执行（正确打包）

    shell>jar cvf0M core.jar BOOT-INF META-INF org

如果能正常启动，则重新打包成功

