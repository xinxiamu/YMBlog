---
title: spring-boot开发常见异常收录
date: 2017-09-29 00:31:25
categories: spring-boot
tags: sboot-often-error
---

## 1. 异常一
异常描述：Cannot determine embedded database driver class for database type NONE

原因：该异常在spring-boot应用启动时候报异常。是因为maven依赖中依赖如了jpa，所以系统会自动配置试图注入jpa数据源。但是如果没又配置数据源，则会报该异常。

### 1.1 处理方法一
在pom中剔除jpa注入

    
### 1.2 在@SpringBootApplication中排除其注入


        @SpringBootApplication(exclude={DataSourceAutoConfiguration.class,HibernateJpaAutoConfiguration.class})


