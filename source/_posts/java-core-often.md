---
title: java-core-often
date: 2019-10-19 10:40:47
categories: java
tags:
---

## java11

#### java11生产jre

java11的jdk下没有jre环境的。想要的话，可以手动生成。    

进入jdk目录下，执行以下命令：

    ## windows
    bin\jlink.exe --module-path jmods --add-modules java.desktop --output jre
    
    ##linux
    bin/jlink --module-path jmods --add-modules java.desktop --output jre
