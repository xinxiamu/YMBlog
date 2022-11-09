---
title: Spring Boot Starter自定义
date: 2022-11-02 14:27:45
categories: spring-boot
tags:
---

在使用spring boot的时候，我们经常能看到很多的`spring-boot-starter-xxx`依赖，这些都是官方定制的`starter`。其有点类似组件、模块的概念，   
你只需要引入相关依赖，甚至不需要做太多什么，就能使用相关功能，很方便很强大是不。这得益于spring boot提供的良好的抽象，遵循的零配置、约定优先的思想。

把常用的共用功能模块抽象成spring boot starter组件，是必须技能。面试也常常会被问。下面我们来自定义自己的starter。

## 自定义starter

官方定义的starter名称一般是`spring-boot-starter-xxx`，那么建议我们自己定义的starter命名则`xxx-spring-boot-starter`。

### 添加依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.springboot.demo</groupId>
	<artifactId>my-spring-boot-starter</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>my-spring-boot-starter</name>
	<packaging>jar</packaging>
	<description>Demo project for Spring Boot</description>
	<properties>
		<java.version>1.8</java.version>
		<spring-boot-dependencies.version>2.2.5.RELEASE</spring-boot-dependencies.version>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
	</properties>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-autoconfigure</artifactId>
			<scope>compile</scope>
		</dependency>

		<!--引入这个依赖，等会你会知道它的作用-->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-configuration-processor</artifactId>
			<optional>true</optional>
		</dependency>

	</dependencies>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-dependencies</artifactId>
				<version>${spring-boot-dependencies.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<build>
		<plugins>
			<!--这个不能用springboot项目默认的，不然install报错-->
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.8.1</version>
				<configuration>
					<source>${java.version}</source>
					<target>${java.version}</target>
					<encoding>${project.build.sourceEncoding}</encoding>
				</configuration>
			</plugin>
		</plugins>
	</build>

</project>
```

### 定义配置类

新建类`PersonProperties`:
```java
package com.example.demo;

import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties(prefix = PersonProperties.PREFIX)
public class PersonProperties {
    
    //配置前缀
    public static final String PREFIX = "ljin-person";

    public PersonProperties(){}

    // 姓名
    private String name;
    // 年龄
    private int age;
    // 性别
    private String sex = "M";

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }
}
```

在引入该starter组件的应用的`application.yaml`配置文件中对应配置该配置类的属性，如果不配置则用的默认值。

### 定义使用上面配置类属性的服务类

新增服务类：`PersonService`
```java
package com.example.demo;

public class PersonService {

    private PersonProperties properties;

    public PersonService() {
    }

    public PersonService(PersonProperties properties) {
        this.properties = properties;
    }

    public void sayHello() {
        String message = String.format("大家好，我叫: %s, 今年 %s岁, 性别: %s",
                properties.getName(), properties.getAge(), properties.getSex());
        System.out.println(message);
    }
}
```

### 定义自动配置类

新增类`PersonServiceAutoConfiguration`：
```java
package com.example.demo;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
@EnableConfigurationProperties(PersonProperties.class)
//当类路径下有指定的类为true
@ConditionalOnClass(PersonService.class)
@ConditionalOnProperty(prefix = PersonProperties.PREFIX, value = "enabled", matchIfMissing = true)
public class PersonServiceAutoConfiguration {

    @Autowired
    private PersonProperties properties;

    // 当容器中没有指定Bean的情况下，自动配置PersonService类
    @Bean
    @ConditionalOnMissingBean(PersonService.class)
    public PersonService personService(){
        PersonService personService = new PersonService(properties);
        return personService;
    }
}
```

注意上面常用注解的使用。

`@ConditionalOnClass(PersonService.class)`: 没有自定义`PersonService`实例Bean的时候，自动配置，否则用自定义的。

`@EnableConfigurationProperties(PersonProperties.class)`: 开启配置。

`@ConditionalOnProperty(prefix = PersonProperties.PREFIX, value = "enabled", matchIfMissing = true)`: 控制是否启动自动配置

**常用注解配置:**

{%asset_img a-1.png%}

### 暴露自动配置类

在`resource`下面新建目录：`META-INF`,新建文件`spring.factories`,内容如下：
```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.example.demo.PersonServiceAutoConfiguration,\
```

> **注意**：其中填的是配置类的全限定名，多个之间逗号分割，还可以 \ 进行转义即相当于去掉后面换行和空格符号

### maven打包配置
```xml
<!--这个不能用springboot项目默认的，不然install报错-->
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.8.1</version>
    <configuration>
        <source>${java.version}</source>
        <target>${java.version}</target>
        <encoding>${project.build.sourceEncoding}</encoding>
    </configuration>
</plugin>
```

## 使用starter

在新项目中操作：

1. 在`pom.xml`中引入依赖：
```xml
<dependency>
    <groupId>com.springboot.demo</groupId>
    <artifactId>my-spring-boot-starter</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</dependency>
```
2. 使用
```java
@RestController
public class StarterTestController {

    @Resource
    private PersonService personService;

    @GetMapping("/sayHello")
    public String sayHello() {
        personService.sayHello();
        return "sayHello";
    }
}
```

上面代码中，`personService`已经自动实例化。

如果想覆盖自动，手动，那只需要定义自己的：
```java
@Configuration
public class MainConfig {

    //如果这里定义，则不会自动定义
   /* @Bean
    public PersonService personService() {
        PersonProperties personProperties = new PersonProperties();
        personProperties.setName("zmttt");
        personProperties.setAge(18);
        personProperties.setSex("男");
        return new PersonService(personProperties);
    }*/
}
```

3. 添加配置给到自动加载类
在使用项目的`application.yml·中：
```yaml
ljin-person:
  sex: 男
  name: zmt
  age: 18
  enabled: true
server:
  port: 8088

```

