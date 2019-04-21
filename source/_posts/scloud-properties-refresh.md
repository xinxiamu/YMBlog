---
title: Spring Cloud外部环境配置刷新
date: 2019-04-20 22:55:37
categories: spring-cloud
tags:
---

本文介绍在spring cloud架构中，如何做配置外部化以及更改配置属性并实时刷新的问题。      
实验环境：
spring boot：2.1.4.RELEASE   
spring cloud: Greenwich.SR1 
github  
rabbitmq: 3.7.8

## 服务注册发现中心eureka-server

关键依赖：

    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
    </dependency>
    
application.yml：

    server:
      port: 1112
    spring:
      application:
        name: eureka-server
      profiles:
        active: dev
    logging:
      level:
        com:
          netflix:
            eureka: off
            discovery: off
      file: logs/${spring.application.name}.log


application-dev.yml：

    eureka:
      instance:
        instance-id: ${spring.application.name}:${spring.cloud.client.ip_address}:${server.port}
        hostname: localhost
      client:
        fetch-registry: false
        register-with-eureka: false  #不注册自己
        service-url:
          defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
      server:
          enable-self-preservation: false #设为false，关闭自我保护
          eviction-interval-timer-in-ms: 4000  # 清理间隔（单位毫秒，默认是60*1000）
    
该服务没什么需要注意的，按正常的来就可以。


## 配置中心config-server    

### 关键依赖：

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-server</artifactId>
    </dependency>
    <!--实时刷新配置 start -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-monitor</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-bus-amqp</artifactId>
    </dependency>
    <!--实时刷新配置 end -->

    <!--作为服务注册到服务注册中心 -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>

要配置刷新，只需要添加：
 
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-monitor</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-bus-amqp</artifactId>
    </dependency>

### 配置

bootstrap.yml：

    server:
      port: 3331
    spring:
      application:
        name: config-server
      profiles:
        active: dev
    ## 非对称加解密,该段配置只能放在bootstrap.yml中
    encrypt:
      key-store:
        location: classpath:/config-server.jks
        password: 111111  # storepass
        alias: config-server    # alias
        secret: 222222   # keypass
    
application.yml：

    management:
      endpoints:
        web:
          exposure:
            include: "*"

`management.endpoints.web.exposure.include`这个配置一定要加上，关键。默认是指暴露站点：info，health。改成星号，代表暴露全部。

application-dev.yml：

    #作为服务注册到注册中心
    eureka:
      server:
        ip: localhost
        port: 1112
      instance:
        instance-id: ${spring.application.name}:${spring.cloud.client.ip-address}:${server.port}
        prefer-ip-address: true #显示ip,浏览器以ip方式请求。
        #域名
    #    hostname: localhost
        #查看健康运行状态
        health-check-url-path: /actuator/health
        status-page-url-path: /actuator/info
      client:
        service-url:
          defaultZone: http://${eureka.server.ip}:${eureka.server.port}/eureka/
    spring:
      security:
          user:
            name: admin
            password: 123456
      cloud:
        #git配置中心
        config:
          server:
            git:
              uri: https://github.com/xrlj/config-repo-dev
              #git的配置文件会加载到本地的目录
              basedir: target/config
              # 设置超时
              timeout: 4
              force-pull: true  #每次都强制拉取远程git的配置更新本地。
              default-label: master
        bus:
          enabled: true
          trace:
            enabled: true
      #          search-paths:
      # spring-cloud-bus刷新配置
      #http://localhost:15672/
      rabbitmq:
          host: 172.31.31.31
          port: 5672
          username: admin
          password: 123456
          publisher-confirms: true
          virtual-host: xr_vhost
    
    encrypt:
      fail-on-error: false
    
注意开启：`spring.cloud.bus.enable=true`,`spring.cloud.bus.trace.enable=true`。

### 温馨提醒

由于添加了安全依赖：

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    
所以config-server的请求是要先登陆验证的。那么，就要要配置`spring-security`，否则后面的`post`请求将会被拒绝。添加配置类如下：    

    package com.xrlj.configserver;
    
    import org.springframework.context.annotation.Configuration;
    import org.springframework.security.config.annotation.web.builders.HttpSecurity;
    import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
    import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
    
    @Configuration
    @EnableWebSecurity
    public class WebSecurityConfiguration extends WebSecurityConfigurerAdapter {
    
        @Override
        protected void configure(HttpSecurity http) throws Exception {
            http.csrf().disable().authorizeRequests()
                    .anyRequest().authenticated().and()
                    .httpBasic();
        }
    }


## 其它业务服务service-sys-common

添加与配置刷新相关依赖：

     <!-- 用户修改git文件，调用接口自动刷新： -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <!--rabbitmq实时刷新配置-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-bus-amqp</artifactId>
    </dependency>
    
关键配置：

    spring:
      cloud:
        bus:
          enabled: true
          trace:
            enabled: true
    #    refresh:
    #      extra-refreshable: javax.sql.DataSource  #该版本中，默认数据源是HikariDataSource，不配置这个，刷新配置报错。
    management:
      endpoints:
        web:
          exposure:
            include: "*"
    
和config-server中一样，星号暴露所有站点，还要bus的一些配置开启。这里有个不同点，`spring.cloud.refresh.extra-refreshable=javax.sql.DataSource`,这里先注释掉，后面再补充说明下。


##  验证配置刷新

1.在`service-sys-common`服务中新建`TestController.java`

    @RefreshScope
    @RestController
    public class TestController {
    
         @Value("${foo}")
         String foo;
    
        @RequestMapping(value = "/hi")
        public String hi(){
            return foo;
        }
    }

注意添加注解`@RefreshScope`。在配置仓库中对应文件添加属性`foo:version1`。 

打开浏览器请求：http://localhost:9010/hi.正常会返回`foo`的值。  

更改foo的值，`foo:version2`,再次请求，看到返回还是version1，证明没刷新。下面刷新下，打开postman发送post刷新请求：localhost:9010/actuator/refresh

然后我们会看到一个错误：    

    2018-02-04 22:11:02.236 DEBUG 6104 --- [           main] o.s.b.d.LoggingFailureAnalysisReporter   : Application failed to start due to an exception
    
    org.springframework.boot.context.properties.bind.BindException: Failed to bind properties under 'spring.datasource.hikari' to com.zaxxer.hikari.HikariDataSource
    	at org.springframework.boot.context.properties.bind.Binder.handleBindError(Binder.java:227) ~[spring-boot-2.0.0.RC1.jar:2.0.0.RC1]
    	at org.springframework.boot.context.properties.bind.Binder.bind(Binder.java:203) ~[spring-boot-2.0.0.RC1.jar:2.0.0.RC1]
    	at org.springframework.boot.context.properties.bind.Binder.bind(Binder.java:187) ~[spring-boot-2.0.0.RC1.jar:2.0.0.RC1]
    	at org.springframework.boot.context.properties.bind.Binder.bind(Binder.java:169) ~[spring-boot-2.0.0.RC1.jar:2.0.0.RC1]
    	at org.springframework.boot.context.properties.ConfigurationPropertiesBinder.bind(ConfigurationPropertiesBinder.java:79) ~[spring-boot-2.0.0.RC1.jar:2.0.0.RC1]
    	at org.springframework.boot.context.properties.ConfigurationPropertiesBindingPostProcessor.postProcessBeforeInitialization(ConfigurationPropertiesBindingPostProcessor.java:167) ~[spring-boot-2.0.0.RC1.jar:2.0.0.RC1]
    	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.applyBeanPostProcessorsBeforeInitialization(AbstractAutowireCapableBeanFactory.java:423) ~[spring-beans-5.0.3.RELEASE.jar:5.0.3.RELEASE]
    	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1702) ~[spring-beans-5.0.3.RELEASE.jar:5.0.3.RELEASE]
    	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:414) ~[spring-beans-5.0.3.RELEASE.jar:5.0.3.RELEASE]
    	at org.springframework.cloud.context.properties.ConfigurationPropertiesRebinder.rebind(ConfigurationPropertiesRebinder.java:101) ~[spring-cloud-context-2.0.0.M5.jar:2.0.0.M5]
    	at org.springframework.cloud.context.properties.ConfigurationPropertiesRebinder.rebind(ConfigurationPropertiesRebinder.java:84) ~[spring-cloud-context-2.0.0.M5.jar:2.0.0.M5]
    	at org.springframework.cloud.context.properties.ConfigurationPropertiesRebinder.onApplicationEvent(ConfigurationPropertiesRebinder.java:132) ~[spring-cloud-context-2.0.0.M5.jar:2.0.0.M5]
    	at org.springframework.cloud.context.properties.ConfigurationPropertiesRebinder.onApplicationEvent(ConfigurationPropertiesRebinder.java:50) ~[spring-cloud-context-2.0.0.M5.jar:2.0.0.M5]
    	at org.springframework.context.event.SimpleApplicationEventMulticaster.doInvokeListener(SimpleApplicationEventMulticaster.java:172) ~[spring-context-5.0.3.RELEASE.jar:5.0.3.RELEASE]
    	at org.springframework.context.event.SimpleApplicationEventMulticaster.invokeListener(SimpleApplicationEventMulticaster.java:165) ~[spring-context-5.0.3.RELEASE.jar:5.0.3.RELEASE]
    	at org.springframework.context.event.SimpleApplicationEventMulticaster.multicastEvent(SimpleApplicationEventMulticaster.java:139) ~[spring-context-5.0.3.RELEASE.jar:5.0.3.RELEASE]
    	at org.springframework.context.support.AbstractApplicationContext.publishEvent(AbstractApplicationContext.java:399) ~[spring-context-5.0.3.RELEASE.jar:5.0.3.RELEASE]
    	at org.springframework.context.support.AbstractApplicationContext.publishEvent(AbstractApplicationContext.java:353) ~[spring-context-5.0.3.RELEASE.jar:5.0.3.RELEASE]
    	at org.springframework.cloud.autoconfigure.ConfigurationPropertiesRebinderAutoConfiguration.afterSingletonsInstantiated(ConfigurationPropertiesRebinderAutoConfiguration.java:77) ~[spring-cloud-context-2.0.0.M5.jar:2.0.0.M5]
    	at org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingletons(DefaultListableBeanFactory.java:778) ~[spring-beans-5.0.3.RELEASE.jar:5.0.3.RELEASE]
    	at org.springframework.context.support.AbstractApplicationContext.finishBeanFactoryInitialization(AbstractApplicationContext.java:868) ~[spring-context-5.0.3.RELEASE.jar:5.0.3.RELEASE]
    	at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:549) ~[spring-context-5.0.3.RELEASE.jar:5.0.3.RELEASE]
    	at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.refresh(ServletWebServerApplicationContext.java:138) ~[spring-boot-2.0.0.RC1.jar:2.0.0.RC1]
    	at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:752) [spring-boot-2.0.0.RC1.jar:2.0.0.RC1]
    	at org.springframework.boot.SpringApplication.refreshContext(SpringApplication.java:388) [spring-boot-2.0.0.RC1.jar:2.0.0.RC1]
    	at org.springframework.boot.SpringApplication.run(SpringApplication.java:327) [spring-boot-2.0.0.RC1.jar:2.0.0.RC1]
    	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1246) [spring-boot-2.0.0.RC1.jar:2.0.0.RC1]
    	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1234) [spring-boot-2.0.0.RC1.jar:2.0.0.RC1]
    	at uk.co.vhome.clubbed.svc.enquiryhandler.ClubbedSvcEnquiryHandlerApplication.main(ClubbedSvcEnquiryHandlerApplication.java:23) [classes/:na]
    Caused by: java.lang.IllegalStateException: Unable to set value for property schema
    	at org.springframework.boot.context.properties.bind.JavaBeanBinder$BeanProperty.setValue(JavaBeanBinder.java:306) ~[spring-boot-2.0.0.RC1.jar:2.0.0.RC1]
    	at org.springframework.boot.context.properties.bind.JavaBeanBinder.bind(JavaBeanBinder.java:76) ~[spring-boot-2.0.0.RC1.jar:2.0.0.RC1]
    	at org.springframework.boot.context.properties.bind.JavaBeanBinder.bind(JavaBeanBinder.java:59) ~[spring-boot-2.0.0.RC1.jar:2.0.0.RC1]
    	at org.springframework.boot.context.properties.bind.JavaBeanBinder.bind(JavaBeanBinder.java:51) ~[spring-boot-2.0.0.RC1.jar:2.0.0.RC1]
    	at org.springframework.boot.context.properties.bind.Binder.lambda$null$5(Binder.java:321) ~[spring-boot-2.0.0.RC1.jar:2.0.0.RC1]
    	at java.base/java.util.stream.ReferencePipeline$3$1.accept(ReferencePipeline.java:195) ~[na:na]
    	at java.base/java.util.ArrayList$ArrayListSpliterator.tryAdvance(ArrayList.java:1471) ~[na:na]
    	at java.base/java.util.stream.ReferencePipeline.forEachWithCancel(ReferencePipeline.java:127) ~[na:na]
    	at java.base/java.util.stream.AbstractPipeline.copyIntoWithCancel(AbstractPipeline.java:502) ~[na:na]
    	at java.base/java.util.stream.AbstractPipeline.copyInto(AbstractPipeline.java:488) ~[na:na]
    	at java.base/java.util.stream.AbstractPipeline.wrapAndCopyInto(AbstractPipeline.java:474) ~[na:na]
    	at java.base/java.util.stream.FindOps$FindOp.evaluateSequential(FindOps.java:152) ~[na:na]
    	at java.base/java.util.stream.AbstractPipeline.evaluate(AbstractPipeline.java:234) ~[na:na]
    	at java.base/java.util.stream.ReferencePipeline.findFirst(ReferencePipeline.java:476) ~[na:na]
    	at org.springframework.boot.context.properties.bind.Binder.lambda$bindBean$6(Binder.java:322) ~[spring-boot-2.0.0.RC1.jar:2.0.0.RC1]
    	at org.springframework.boot.context.properties.bind.Binder$Context.withIncreasedDepth(Binder.java:415) ~[spring-boot-2.0.0.RC1.jar:2.0.0.RC1]
    	at org.springframework.boot.context.properties.bind.Binder$Context.withBean(Binder.java:405) ~[spring-boot-2.0.0.RC1.jar:2.0.0.RC1]
    	at org.springframework.boot.context.properties.bind.Binder.bindBean(Binder.java:319) ~[spring-boot-2.0.0.RC1.jar:2.0.0.RC1]
    	at org.springframework.boot.context.properties.bind.Binder.bindObject(Binder.java:261) ~[spring-boot-2.0.0.RC1.jar:2.0.0.RC1]
    	at org.springframework.boot.context.properties.bind.Binder.bind(Binder.java:198) ~[spring-boot-2.0.0.RC1.jar:2.0.0.RC1]
    	... 27 common frames omitted
    Caused by: java.lang.reflect.InvocationTargetException: null
    	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:na]
    	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:na]
    	at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:na]
    	at java.base/java.lang.reflect.Method.invoke(Method.java:564) ~[na:na]
    	at org.springframework.boot.context.properties.bind.JavaBeanBinder$BeanProperty.setValue(JavaBeanBinder.java:303) ~[spring-boot-2.0.0.RC1.jar:2.0.0.RC1]
    	... 46 common frames omitted
    Caused by: java.lang.IllegalStateException: The configuration of the pool is sealed once started.  Use HikariConfigMXBean for runtime changes.
    	at com.zaxxer.hikari.HikariConfig.setSchema(HikariConfig.java:951) ~[HikariCP-2.7.6.jar:na]
    	... 51 common frames omitted
    
    2018-02-04 22:11:02.237 ERROR 6104 --- [           main] o.s.b.d.LoggingFailureAnalysisReporter   : 
    
    ***************************
    APPLICATION FAILED TO START
    ***************************
    
    Description:
    
    Failed to bind properties under 'spring.datasource.hikari' to com.zaxxer.hikari.HikariDataSource:
    
        Property: spring.datasource.hikari.schema
        Value: ${application.database.schema}
        Origin: class path resource [application.properties]:11:33
        Reason: Unable to set value for property schema
    
    Action:
    
    Update your application's configuration
    
    
    Process finished with exit code 1
    
爆出该异常源码地方：类ConfigurationPropertiesRebinder

    @ManagedOperation
    public boolean rebind(String name) {
        if (!this.beans.getBeanNames().contains(name)) {
            return false;
        }
        if (this.applicationContext != null) {
            try {
                Object bean = this.applicationContext.getBean(name);
                if (AopUtils.isAopProxy(bean)) {
                    bean = ProxyUtils.getTargetObject(bean);
                }
                if (bean != null) {
                    this.applicationContext.getAutowireCapableBeanFactory()
                            .destroyBean(bean);
                    this.applicationContext.getAutowireCapableBeanFactory()
                            .initializeBean(bean, name);
                    return true;
                }
            }
            catch (RuntimeException e) {
                this.errors.put(name, e);
                throw e;
            }
            catch (Exception e) {
                this.errors.put(name, e);
                throw new IllegalStateException("Cannot rebind to " + name, e);
            }
        }
        return false;
    }    
    
 问题原因：  
 那是因为，我们在项目里自定义了动态数据源，采用了默认的数据库连接池HikariDataSource，该连接池，一旦创建，讲不可更改，所以当你刷新配置的时候，是一起连同数据源的相关配置也要刷新的，所以报错了。    
 数据源配置如下：   
 
    package com.xrlj.framework.spring.config.ds.myself;
    
    import com.xrlj.framework.dao.ds.DynamicDataSource;
    import com.zaxxer.hikari.HikariDataSource;
    import org.springframework.beans.factory.annotation.Qualifier;
    import org.springframework.boot.context.properties.ConfigurationProperties;
    import org.springframework.boot.jdbc.DataSourceBuilder;
    import org.springframework.context.annotation.*;
    
    /**
     * 按规则定死的数据源。一主双从。适用所有服务。
     */
    @Configuration
    public class DataSourceConfig {
    
        @Bean(name = "masterDataSource")
        @Qualifier("masterDataSource")
        @ConfigurationProperties(prefix="spring.myself-db.datasource.hikari.master")
        public DataSource masterDataSource() {
            return DataSourceBuilder.create().type(HikariDataSource.class).build();
        }
    
        @Bean(name = "slave1DataSource")
        @Qualifier("slave1DataSource")
        @ConfigurationProperties(prefix="spring.myself-db.datasource.hikari.slave1")
        public DataSource slave1DataSource() {
            return DataSourceBuilder.create().type(HikariDataSource.class).build();
        }
    
        @Bean(name = "slave2DataSource")
        @Qualifier("slave2DataSource")
        @ConfigurationProperties(prefix="spring.myself-db.datasource.hikari.slave2")
        public DataSource slave2DataSource() {
            return DataSourceBuilder.create().type(HikariDataSource.class).build();
        }
    
        /**
         * 动态数据源: 通过AOP在不同数据源之间动态切换
         *
         * @return
         */
        @Primary
        @Bean(name = "dynamicDataSource")
        @Qualifier("dynamicDataSource")
        @Scope("singleton")
        @DependsOn({"masterDataSource","slave1DataSource","slave2DataSource"}) //要加入这个注解，在数据源初始化之后，再初始化本bean，否则会出现循环依赖注入无法启动。
        public DataSource dynamicDataSource(@Qualifier("masterDataSource") DataSource masterDataSource,
                                            @Qualifier("slave1DataSource") DataSource slave1DataSource,@Qualifier("slave2DataSource") DataSource slave2DataSource) {
            DynamicDataSource dynamicDataSource = new DynamicDataSource();
            return dynamicDataSource.setMultipleDataSource(masterDataSource,slave1DataSource,slave2DataSource);
        }
    }

折腾了半天，终于在官网wiki中找到答案。   
参考：     
https://github.com/spring-cloud/spring-cloud-commons/pull/395   
https://github.com/spring-cloud/spring-cloud-commons/issues/318 

其中有一段问题解决的描述如下： 

After trying to implement this, we are going to add a documentation note, but you should either set spring.cloud.refresh.extra-refreshable=javax.sql.DataSource or, more appropriately, strongly type your DataSource bean

    @Primary
    @Bean(name = "dbDataSource")
    @ConfigurationProperties(prefix = "datasource.db")
    public HikariDataSource dbDataSource() {
        return DataSourceBuilder.create().type(HikariDataSource.class).build();
    }

意思是说，解决该问题有两种方式：    

1.添加配置：`spring.cloud.refresh.extra-refreshable=javax.sql.DataSource`    
2.配置数据源时，指定最终的类型，而不是接口类型。如下：    
把  
 
    @Bean(name = "masterDataSource")
    @Qualifier("masterDataSource")
    @ConfigurationProperties(prefix="spring.myself-db.datasource.hikari.master")
    public DataSource masterDataSource() {
        return DataSourceBuilder.create().type(HikariDataSource.class).build();
    }
    
改成：     

    @Bean(name = "masterDataSource")
    @Qualifier("masterDataSource")
    @ConfigurationProperties(prefix="spring.myself-db.datasource.hikari.master")
    public DataSource masterDataSource() {
        return DataSourceBuilder.create().type(HikariDataSource.class).build();
    }

每个地方都改成明确的类型。   

如果采用第一种方式，会出现另外一个问题，在后面代码中DynamicDataSource转型为DataSource将会报错。纳闷，狗日的……

因此，只能采用方式二了。把数据源配置改如下：

    package com.xrlj.framework.spring.config.ds.myself;
    
    import com.xrlj.framework.dao.ds.DynamicDataSource;
    import com.zaxxer.hikari.HikariDataSource;
    import org.springframework.beans.factory.annotation.Qualifier;
    import org.springframework.boot.context.properties.ConfigurationProperties;
    import org.springframework.boot.jdbc.DataSourceBuilder;
    import org.springframework.context.annotation.*;
    
    /**
     * 按规则定死的数据源。一主双从。适用所有服务。
     */
    @Configuration
    public class DataSourceConfig {
    
        @Bean(name = "masterDataSource")
        @Qualifier("masterDataSource")
        @ConfigurationProperties(prefix="spring.myself-db.datasource.hikari.master")
        public HikariDataSource masterDataSource() {
            return DataSourceBuilder.create().type(HikariDataSource.class).build();
        }
    
        @Bean(name = "slave1DataSource")
        @Qualifier("slave1DataSource")
        @ConfigurationProperties(prefix="spring.myself-db.datasource.hikari.slave1")
        public HikariDataSource slave1DataSource() {
            return DataSourceBuilder.create().type(HikariDataSource.class).build();
        }
    
        @Bean(name = "slave2DataSource")
        @Qualifier("slave2DataSource")
        @ConfigurationProperties(prefix="spring.myself-db.datasource.hikari.slave2")
        public HikariDataSource slave2DataSource() {
            return DataSourceBuilder.create().type(HikariDataSource.class).build();
        }
    
        /**
         * 动态数据源: 通过AOP在不同数据源之间动态切换
         *
         * @return
         */
        @Primary
        @Bean(name = "dynamicDataSource")
        @Qualifier("dynamicDataSource")
        @Scope("singleton")
        @DependsOn({"masterDataSource","slave1DataSource","slave2DataSource"}) //要加入这个注解，在数据源初始化之后，再初始化本bean，否则会出现循环依赖注入无法启动。
        public DynamicDataSource dynamicDataSource(@Qualifier("masterDataSource") HikariDataSource masterDataSource,
                                            @Qualifier("slave1DataSource") HikariDataSource slave1DataSource,@Qualifier("slave2DataSource") HikariDataSource slave2DataSource) {
            DynamicDataSource dynamicDataSource = new DynamicDataSource();
            return dynamicDataSource.setMultipleDataSource(masterDataSource,slave1DataSource,slave2DataSource);
        }
    
    }

再重新启动各服务，测试：    
改动foo的值，刷新页面（post）： localhost:9010/actuator/refresh：

{% asset_img a-1.png %}

再请求`/hi`:   

{% asset_img a-2.png %}

我们看到，foo的值已经在线刷新。无需重启服务。    

如果是多个实例，只要任何一个实例执行了刷新请求，所有实例都会同步刷新的。


## 统一在config-server端刷新配置

上面我们的测试，都是对每个业务服务进行配置刷新的。如果业务服务很多的话，就做不到统一刷新了，所以我们这里介绍在配置中心端刷新配置。   

流程是，请求config-server的刷新请求，它会向mq发送配置刷新通知，然后，所有参与订阅的服务的配置都会收到刷新的通知时间，然后自动刷新。当然，也可以指定只刷新某个服务的。  

更改foo的值，发起刷新请求，注意，请求的是config-server的地址：

http://localhost:3331/actuator/bus-refresh   

注意请求地址，是`bus-refresh`，而不是`refresh`。否则不成功。

{%asset_img b-1.png%}

这里要注意下，请求里面要添加认证信息。因为config-server里面引入了`spring-security`模块，而且要添加安全配置`WebSecurityConfiguration`,否则会出现请求需要验证或者请求拒绝导致不成功。  

看请求结果图，请求返回状态为204 No Content,显示请求成功，但是没有返回内容。不知道为啥……

再次请求业务服务的`/hi`,可以看到，配置已经在线刷新。成功了！

## 利用github的Webhooks功能自动刷新

达到的目的是，改动配置后，提交到git仓库，然后触发事件发起请求config-server的刷新请求。达到自动目的。

webhooks的使用，这里不做介绍。

略……
