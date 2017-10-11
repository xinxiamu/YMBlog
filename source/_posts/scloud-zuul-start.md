---
title: spring-cloud-zuul网关入门
date: 2017-10-11 15:42:53
categories: spring-cloud
tags: zuul-start
---
## 1. Zuul简介
路由是微服务架构的不可或缺的一部分。例如：”/” 可能映射到你应用主页，/api/users映射到用户服务，/api/shop映射到购物服务。Zuul。Zuul是Netflix出品的一个基于JVM路由和服务端的负载均衡器。

能做什么：
> - Authentication
- Insights
- Stress Testing
- Canary Testing
- Dynamic Routing
- Service Migration
- Load Shedding
- Security
- Static Response handling
- Active/Active traffic management

引入网关后，整个微服务架构演变为:

{% asset_img a.png %} 

## 2. 使用Zuul

### 2.1 引入Zuul组件

    <dependencies>
        <!--引入网关组件-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zuul</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>
    
        <!--读取配置中心-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
    </dependencies>

### 2.2 开启Zuul
    
    /**
     * 使用@EnableZuulProxy注解激活zuul。
     * 跟进该注解可以看到该注解整合了@EnableCircuitBreaker、@EnableDiscoveryClient，是个组合注解，目的是简化配置。
     */
    @EnableZuulProxy
    @SpringBootApplication
    public class SpcsellingApiGatewayApp {
    
    	public static void main(String[] args) {
    		new SpringApplicationBuilder(SpcsellingApiGatewayApp.class).run(args);
    	}
    }	
    
### 2.3 添加基本配置(.yml )   

    spring:
      application:
        name: api-gateway
    
    ---
    eureka:
      instance:
        hostname: api.spcs.com #域名
      client:
        service-url:
           defaultZone: http://localhost:1111/eureka/ #注册发现服务
    
    ---
    #从配置中心读取配置
    spring:
      cloud:
        config:
          name: api-gateway
          profile: dev
          label: master
          fail-fast: true
          discovery:
            enabled: true
            service-id: config-server
          username: admin
          password: 123456
          
    #配置路由
    zuul:
      routes:
        api-a:
          path: /a/**
          stripPrefix: true
          service-id: service-a #服务id  
        api-b:
          path: /b/**
          stripPrefix: true
          service-id: service-b #服务id
          
首先向eureka注册自己，服务名称为api-gateway；请求路由示例：api.spcs.com/a/users/1 将路由到服务service-a,为：localhost:8001/users/1。b服务的路由也类似。

## 3. 具体配置使用

### 3.1 负载均衡访问服务

*application.yml.*
 
    zuul:
      routes:
        users:
          path: /myusers/**
          serviceId: users
    
    # 关闭ribbon负载均衡器
    ribbon:
      eureka:
        enabled: false
    
    #user服务
    users: 
      ribbon:
        listOfServers: example.com,google.com   #多个实例      