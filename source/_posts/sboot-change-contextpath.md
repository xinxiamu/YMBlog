---
title: spring-boot更改ContextPath方法
date: 2017-09-28 14:03:23
categories: spring-boot
tags: change-contextpath-way
---
在spring-boot项目中，启动后容器的默认context-path为/,如：`http://localhost:8080/`，那么如何改成`http://localhost:8080/api`的形式呢？有以下几种方式：

## 1. 在配置文件Properties & Yaml添加配置

### 1.1 更改properties 
    /src/main/resources/application.properties
    server.port=8080
    server.contextPath=/mkyong

### 1.2 更改yaml
    /src/main/resources/application.properties
    server:
      port: 8080
      contextPath: /mkyong
      
## 2、 自定义容器设置EmbeddedServletContainerCustomizer

`CustomContainer.java`
  
    package com.mkyong;
    
    import org.springframework.boot.context.embedded.ConfigurableEmbeddedServletContainer;
    import org.springframework.boot.context.embedded.EmbeddedServletContainerCustomizer;
    import org.springframework.stereotype.Component;
    
    @Component
    public class CustomContainer implements EmbeddedServletContainerCustomizer {
    
    	@Override
    	public void customize(ConfigurableEmbeddedServletContainer container) {
    
    		container.setPort(8080);
    		container.setContextPath("/mkyong");
    
    	}
    
    }   
    
## 3. 命令行方式
`java -jar -Dserver.contextPath=/mkyong spring-boot-example-1.0.jar`
    