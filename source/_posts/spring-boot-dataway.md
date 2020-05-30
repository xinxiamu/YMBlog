---
title: Spring Boot整合神器-Dataway
date: 2020-05-30 22:17:05
categories: spring-boot
tags:
---

多年前，当我还是个安卓开发小哥的时候，后台给了我一个神奇的接口对接，参数只需要传表名以及表中你需要的字段名，接口就执行该查询语句，并返回你想要的结果……
那会，那后台哥哥剩下的就只负责喝茶聊天了，而我，内心有一万个草泥马。  

后来，自己做后台开发了，心中会偶尔想，能不能只需要写一个接口或者一个框架，让前端自己去定义接口，按需去取。但是能力有限，始终无法付诸行动……

然而今天，偶然看到一神器名叫Dataway，简单了解后，突然如获至宝，我去……

赶紧记录下来……

## 介绍

_1.优势_:

按照某种固定模式，自由定义接口，并按照定义的sql语句，即可得到查询结果。       
简单点来说，即把sql查询结果直接转换成http接口对外提供。 

_2.缺点_：

由于返回结果通过直接的sql查询，因此想要得到复杂的返回值，比如跨越多张表并且要对结果进行各种再处理的情况，则难以达到要求。

_3.适用：_

在一些中小企业，技术团队比较弱小甚至只有几个运维的情况下，当他们想要对外开放erp的数据时候，他们是怎么做的呢？    

我见过方法之一就是，建立erp数据库视图供外部使用。考虑到erp数据的安全，且只提供只读视图。

突然发觉，开放视图的方式，和这里介绍的Dataway有着异曲同工之妙。好像都是一样一样的！偷笑……

很炸裂，有一个运维，够了，只需要写sql就行了，外部系统就能通过http接口拿到结果集。

## 集成Spring Boot

#### 引入相关依赖

新建spring-boot项目后，pom.xml中引入依赖：

```text
<!-- 引入依赖 -->
<dependency>
    <groupId>net.hasor</groupId>
    <artifactId>hasor-spring</artifactId>
    <version>4.1.7</version><!-- 查看最新版本：https://mvnrepository.com/artifact/net.hasor/hasor-spring -->
</dependency>
<dependency>
    <groupId>net.hasor</groupId>
    <artifactId>hasor-dataway</artifactId>
    <version>4.1.7</version><!-- 查看最新版本：https://mvnrepository.com/artifact/net.hasor/hasor-dataway -->
</dependency>
```
   
#### 配置文件中启用Dataway

在应用的 application.properties 配置文件中启用 Dataway

```properties
# 启用 Dataway 功能（默认不启用）
HASOR_DATAQL_DATAWAY=true
# 开启 ui 管理功能（注意生产环境必须要设置为 false，否则会造成严重的生产安全事故）
HASOR_DATAQL_DATAWAY_ADMIN=true

# （可选）API工作路径
HASOR_DATAQL_DATAWAY_API_URL=/api/
# （可选）ui 的工作路径，只有开启 ui 管理功能后才有效
HASOR_DATAQL_DATAWAY_UI_URL=/interface-ui/
```


    
#### 初始化必要的表(例：MySQL)

```sql
CREATE TABLE `interface_info` (
    `api_id`          int(11)      NOT NULL AUTO_INCREMENT   COMMENT 'ID',
    `api_method`      varchar(12)  NOT NULL                  COMMENT 'HttpMethod：GET、PUT、POST',
    `api_path`        varchar(512) NOT NULL                  COMMENT '拦截路径',
    `api_status`      int(2)       NOT NULL                  COMMENT '状态：0草稿，1发布，2有变更，3禁用',
    `api_comment`     varchar(255)     NULL                  COMMENT '注释',
    `api_type`        varchar(24)  NOT NULL                  COMMENT '脚本类型：SQL、DataQL',
    `api_script`      mediumtext   NOT NULL                  COMMENT '查询脚本：xxxxxxx',
    `api_schema`      mediumtext       NULL                  COMMENT '接口的请求/响应数据结构',
    `api_sample`      mediumtext       NULL                  COMMENT '请求/响应/请求头样本数据',
    `api_option`      mediumtext       NULL                  COMMENT '扩展配置信息',
    `api_create_time` datetime     DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    `api_gmt_time`    datetime     DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
    PRIMARY KEY (`api_id`)
) ENGINE=InnoDB AUTO_INCREMENT=0 DEFAULT CHARSET=utf8mb4 COMMENT='Dataway 中的API';

CREATE TABLE `interface_release` (
    `pub_id`          int(11)      NOT NULL AUTO_INCREMENT   COMMENT 'Publish ID',
    `pub_api_id`      int(11)      NOT NULL                  COMMENT '所属API ID',
    `pub_method`      varchar(12)  NOT NULL                  COMMENT 'HttpMethod：GET、PUT、POST',
    `pub_path`        varchar(512) NOT NULL                  COMMENT '拦截路径',
    `pub_status`      int(2)       NOT NULL                  COMMENT '状态：0有效，1无效（可能被下线）',
    `pub_type`        varchar(24)  NOT NULL                  COMMENT '脚本类型：SQL、DataQL',
    `pub_script`      mediumtext   NOT NULL                  COMMENT '查询脚本：xxxxxxx',
    `pub_script_ori`  mediumtext   NOT NULL                  COMMENT '原始查询脚本，仅当类型为SQL时不同',
    `pub_schema`      mediumtext       NULL                  COMMENT '接口的请求/响应数据结构',
    `pub_sample`      mediumtext       NULL                  COMMENT '请求/响应/请求头样本数据',
    `pub_option`      mediumtext       NULL                  COMMENT '扩展配置信息',
    `pub_release_time`datetime     DEFAULT CURRENT_TIMESTAMP COMMENT '发布时间（下线不更新）',
    PRIMARY KEY (`pub_id`)
) ENGINE=InnoDB AUTO_INCREMENT=0 DEFAULT CHARSET=utf8mb4 COMMENT='Dataway API 发布历史。';

create index idx_interface_release on interface_release (pub_api_id);
```

#### 初始化数据源

如果项目中已经配置了数据源，则直接将 Spring 使用的数据源导入到 Hasor 环境共 Dataway 使用。

```java
@DimModule
@Component
public class ExampleModule implements SpringModule {
    @Autowired
    private DataSource dataSource = null;

    @Override
    public void loadModule(ApiBinder apiBinder) throws Throwable {
        // .DataSource form Spring boot into Hasor
        apiBinder.installModule(new JdbcModule(Level.Full, this.dataSource));
        // .custom DataQL
        //apiBinder.tryCast(QueryApiBinder.class).loadUdfSource(apiBinder.findClass(DimUdfSource.class));
        //apiBinder.tryCast(QueryApiBinder.class).bindFragment("sql", SqlFragment.class);
    }
}
```

如果没有配置数据源，则要配置数据源：

1.引入依赖

```text
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.1.10</version>
</dependency>
```

2.添加配置

在application.properties中添加数据源配置：

```properties
# db
spring.datasource.url=jdbc:mysql://172.31.31.31:3910/dataway?useUnicode=true&characterEncoding=utf8&useTimezone=true&serverTimezone=Asia/Shanghai
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
# druid
spring.datasource.druid.initial-size=3
spring.datasource.druid.min-idle=3
spring.datasource.druid.max-active=10
spring.datasource.druid.max-wait=60000
spring.datasource.druid.stat-view-servlet.login-username=admin
spring.datasource.druid.stat-view-servlet.login-password=admin
spring.datasource.druid.filter.stat.log-slow-sql=true
spring.datasource.druid.filter.stat.slow-sql-millis=1
```

Hasor 启动的时候会调用 loadModule 方法，在这里再把 DataSource 设置到 Hasor 中。

#### 在SprintBoot 中启用 Hasor

```java
import net.hasor.spring.boot.EnableHasor;
import net.hasor.spring.boot.EnableHasorWeb;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@EnableHasor
@EnableHasorWeb
@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

    @Controller
    @ResponseBody
    @RequestMapping("/")
    public class HomeController {

        /**
         * 输入http://localhost:8080直接跳转到http://localhost:8080/interface-ui/
         * @param request
         * @param response
         * @throws IOException
         */
        @GetMapping
        public void index(HttpServletRequest request, HttpServletResponse response) throws IOException {
           response.sendRedirect("/interface-ui/");
        }
    }
}
```

这一步非常简单，只需要在 Spring 启动类上增加两个注解即可。

启动后，浏览器打开http://localhost:8080。

具体使用参考：

[https://www.hasor.net/web/dataway/for_boot.html](https://www.hasor.net/web/dataway/for_boot.html)