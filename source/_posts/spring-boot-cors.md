---
title: springboot跨域请求解决
date: 2017-11-15 09:43:12
categories: spring-boot
tags: spring-cors
---
先推荐三篇文章
[跨域资源共享 CORS 详解](http://www.ruanyifeng.com/blog/2016/04/cors.html)
[同源策略和跨域访问](http://blog.csdn.net/shimiso/article/details/21830313)
[详解js跨域问题](https://segmentfault.com/a/1190000000718840)

## 什么是跨域

要想了解跨域，就先要知道什么是同源策略。

同源策略，它是由Netscape提出的一个著名的安全策略。

同源策略（Same origin policy）是一种约定，它是浏览器最核心也最基本的安全功能，如果缺少了同源策略，则浏览器的正常功能可能都会受到影响。可以说Web是构建在同源策略基础之上的，浏览器只是针对同源策略的一种实现。

域：协议+地址(域名或IP)+端口

## 为什么要有同源策略

可以简单的理解为：同源策略是一个安全策略，浏览器只是对同源策略的一种实现。它限制着只有同源的脚本(Javascript)才能调用该源的接口，以保护服务器资源或数据。

## 为什么要有跨域

最常见的：多个系统前端需要调用另外系统的接口；前后端分开部署。

## 如何解决跨域

*JSONP*

只能实现GET请求，但是被一些老浏览器支持。

*代理*

在服务器端处理其他源资源请求访问，使得浏览器端无跨域问题。

*CORS*

2014年1月16日，W3C的Web应用工作组（Web Applications Working Group）和Web应用安全工作组（Web AppSec）联合发布了跨源资源共享（Cross-Origin Resource Sharing）的W3C正式推荐标准（W3C Recommendation）。该标准定义了在必须访问跨域资源时，浏览器与服务端应该如何沟通，它提供一种机制，允许客户端（如浏览器）对非源站点的资源发出访问请求。所有提供跨源资源请求的API都可以使用本规范中定义的算法。

出于安全性的考虑，用户代理（如浏览器）通常拒绝跨站的访问请求，但这会限制运行在用户代理的Web应用通过Ajax或者其他机制从另一个站点访问资源、获取数据。跨源资源共享（CORS）扩充了这个模型，通过使用自定义的HTTP响应头部（HTTP Response Header），通知浏览器资源可能被哪些跨源站点以何种HTTP方法获得。例如，浏览器在访问 http://example.com 站点的Web应用时，Web应用如果需要跨站访问另一站点的资源 http://hello-world.example，就需要使用该标准。http://hello-world.example 在HTTP的响应头部中定义 Access-Control-Allow-Origin: http://example.org，通知浏览器允许 http://example.org 跨源从 http://hello-world.example上获取资源。

## springboot跨域

### 设置全局跨域
- 方法一：
    
        @Configuration
        public class WebConfig extends WebMvcConfigurationSupport {
            /**
             * 全局跨域设置
             *
             * @param registry
             */
            @Override
            protected void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/**")
                        //放行哪些原始域
                        .allowedOrigins("*")
                        //是否发送Cookie信息
                        .allowCredentials(true)
                        //放行哪些原始域(请求方式)
                        .allowedMethods("GET", "POST", "PUT", "DELETE")
                        //放行哪些原始域(头部信息)
                        .allowedHeaders("*");
        //                //暴露哪些头部信息（因为跨域访问默认不能获取全部头部信息）
        //                .exposedHeaders("Header1", "Header2");
            }
        
        }

### 局部跨域
