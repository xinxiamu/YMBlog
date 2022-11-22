---
title: sharding-jdbc-start
date: 2022-11-17 12:45:24
categories:
tags:
---

这里只介绍记录`sharding-jdb`各个核心功能的使用方式，不对概念、原理做过多介绍。想要了解相关概念、术语、原理，请查看[官网](https://shardingsphere.apache.org/index_zh.html)。    
当然，在使用之前，如果你还没对`sharding-jdbc`相关了解，建议您先打开官网先了解其相关概念以及原理。

集成环境：spring boot 2.7.5 + shardingsphere 5.2.1 + mybatis + mysql8

参考： https://forezp.blog.csdn.net/article/details/94343671?spm=1001.2101.3001.6650.4&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-4-94343671-blog-121531810.pc_relevant_aa2&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-4-94343671-blog-121531810.pc_relevant_aa2&utm_relevant_index=5

## 读写分离

1. 添加jar依赖
```xml
<dependency>
    <groupId>org.apache.shardingsphere</groupId>
    <artifactId>shardingsphere-jdbc-core-spring-boot-starter</artifactId>
    <version>5.2.1</version>
</dependency>
<dependency>
    <groupId>org.yaml</groupId>
    <artifactId>snakeyaml</artifactId>
    <version>1.33</version>
</dependency>
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.2.2</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```

2. 静态读写分离配置

新建配置文件`application-readwrite-dynamic.properties`，内容如下：

```properties
#spring.shardingsphere.mode.type=Standalone
#数据源配置 start

#数据源别名，下面配置使用
spring.shardingsphere.datasource.names=ds-0,ds-1,ds-2

spring.shardingsphere.datasource.ds-0.jdbc-url=jdbc:mysql://192.168.0.106:3910/cool?serverTimezone=UTC&useSSL=false&useUnicode=true&characterEncoding=UTF-8
spring.shardingsphere.datasource.ds-0.type=com.zaxxer.hikari.HikariDataSource
spring.shardingsphere.datasource.ds-0.driver-class-name=com.mysql.jdbc.Driver
spring.shardingsphere.datasource.ds-0.username=root
spring.shardingsphere.datasource.ds-0.password=123456
spring.shardingsphere.datasource.ds-0.maxPoolSize=20

spring.shardingsphere.datasource.ds-1.jdbc-url=jdbc:mysql://192.168.0.106:3911/cool?serverTimezone=UTC&useSSL=false&useUnicode=true&characterEncoding=UTF-8
spring.shardingsphere.datasource.ds-1.type=com.zaxxer.hikari.HikariDataSource
spring.shardingsphere.datasource.ds-1.driver-class-name=com.mysql.jdbc.Driver
spring.shardingsphere.datasource.ds-1.username=root
spring.shardingsphere.datasource.ds-1.password=123456
spring.shardingsphere.datasource.ds-1.maxPoolSize=20

spring.shardingsphere.datasource.ds-2.jdbc-url=jdbc:mysql://192.168.0.106:3912/cool?serverTimezone=UTC&useSSL=false&useUnicode=true&characterEncoding=UTF-8
spring.shardingsphere.datasource.ds-2.type=com.zaxxer.hikari.HikariDataSource
spring.shardingsphere.datasource.ds-2.driver-class-name=com.mysql.jdbc.Driver
spring.shardingsphere.datasource.ds-2.username=root
spring.shardingsphere.datasource.ds-2.password=123456
spring.shardingsphere.datasource.ds-2.maxPoolSize=20

#数据源配置 end

#静态读写分离
spring.shardingsphere.rules.readwrite-splitting.data-sources.readwrite_ds.static-strategy.write-data-source-name=ds-0
#从库，多个逗号隔开
spring.shardingsphere.rules.readwrite-splitting.data-sources.readwrite_ds.static-strategy.read-data-source-names=ds-1,ds-2
#负载均衡算法,这里用的平均轮询，其他的请查看官网说明
spring.shardingsphere.rules.readwrite-splitting.data-sources.readwrite_ds.load-balancer-name=round_robin
spring.shardingsphere.rules.readwrite-splitting.load-balancers.round_robin.type=ROUND_ROBIN

#是否打印显示sql日志到控制台。生产环境建议关闭，设置成false
spring.shardingsphere.props.sql-show=true

```

静态读写分离不是高可用的，当有从库宕机后，程序并没有主动切断并调整数据源关系，导致还是分连接请求到该宕机的mysql服务。
所以，想要自动管理这些，就要引入高可用和动态读写的功能。

配置文件`application.yml`内容如下：
```yaml
mybatis:
  config-location: classpath:mybatis-config.xml
server:
  port: 8081
spring:
  profiles:
    active: readwrite-dynamic
```

项目结构：

{%asset_img a-2.png%}

3. 高可用和动态数据源

4. 测试实验

{%asset_img a-1.png%}


## 分表

## 分库
