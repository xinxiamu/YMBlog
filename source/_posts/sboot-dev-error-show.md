---
title: spring-boot开发常见异常收录
date: 2017-09-29 00:31:25
categories: spring-boot
tags: sboot-often-error
---

## 1. 异常一：缺少jpa数据源配置
异常描述：Cannot determine embedded database driver class for database type NONE

原因：该异常在spring-boot应用启动时候报异常。是因为maven依赖中依赖如了jpa，所以系统会自动配置试图注入jpa数据源。但是如果没又配置数据源，则会报该异常。

### 1.1 处理方法一
在pom中剔除jpa注入

     <dependency>
         <groupId>com.ymu.spcselling</groupId>
         <artifactId>spcselling-infrastructure</artifactId>
         <exclusions>
             <exclusion>
                 <groupId>org.springframework.boot</groupId>
                 <artifactId>spring-boot-starter-data-jpa</artifactId>
             </exclusion>
         </exclusions>
     </dependency>
     
### 1.2　不传递依赖
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
        <optional>true</optional><!--不在子应用中传递-->
    </dependency>    

    
### 1.3 在@SpringBootApplication中排除其注入
    @SpringBootApplication(exclude={DataSourceAutoConfiguration.class,HibernateJpaAutoConfiguration.class})

