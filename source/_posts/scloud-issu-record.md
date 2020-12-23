---
title: Spring Cloud使用问题记录
date: 2019-05-31 18:20:21
categories: spring-cloud
tags:
---

## Zuul问题

1、max-threads：需要根据具体的硬件环境进行调整
2、max-threads：并不是线程数越大越好，线程数增加会增加内存开销同时大量线程切换会浪费不少时间，并且容易造成内存溢出
3、如果单一网关不能满足并发量，网关可以集群，使用nginx分发请求给多个网关
4、可以使用spring-session、redis解决session一致性问题

#### header信息路由到服务后丢失

解决问题
 
- 保留请求header信息。全部配置
zuul.sensitive-headers=
或者具体服务配置
zuul.routes.xxx.sensitive-headers=
zuul.routes.xxx.custom-sensitive-headers=true

参考这篇文章

Spring Cloud实战小贴士：Zuul处理Cookie和重定向

http://blog.csdn.net/dream8062/article/details/71169628

http://blog.didispace.com/spring-cloud-zuul-cookie-redirect/

#### 跨域问题

_问题描述：_

正常情况下，跨域是这样的：
1. 微服务配置跨域+zuul不配置=有跨域问题
2. 微服务配置+zuul配置=有跨域问题
3. 微服务不配置+zuul不配置=有跨域问题
4. 微服务不配置+zuul配置=ok

然而云环境中每个服务自己有跨域解决方案，而网关需要做最外层的跨域解决方案.如果服务已有跨域配置网关也有，会出现*多次配置问题。

    Access-Control-Allow-Origin:"*,*"
    
    也就是multiple Access-Control-Allow-Origin
    
！！！所以我们就要，微服务配置+zuul配置=解决跨域问题

_问题解决：_

使用ZUUL配置忽略头部信息

    zuul:
      #需要忽略的头部信息，不再传播到其他服务。
      ignored-headers: Access-Control-Allow-Origin,H-APP-Id,Token,APPToken    


#### cookie无法跨域，会话无法保持的问题

_问题：_

当我们解决了上面的头路由到服务，跨域的问题后。但是又出现了另外的问题，浏览器无法保存cookie信息，导致每次发起请求，都会重新创建不同的session会话。 
这样的话，客户端频繁的请求将会在服务端创建大量的session对象，这对服务器是个很大的负担，会话无法保持，每次都新建会话。

_解决：_

1.在zuul网关层更改跨域配置：

    # cors跨域设置
    custom:
      cors:
        mapping: /**
        allowCredentials: true #允许cookie跨域
        allowedOrigins: "*" #允许的域，多个用逗号隔开。*允许全部的域通过。一定要用双引号，否则配置文件报错。
        #    allowedMethods: POST,GET,DELETE,PUT #这样设置，静态资源将不能跨域
        allowedMethods: "*"
        allowedHeaders: "*"
        
关键是：`allowCredentials: true`,设置为true，即允许cookie跨域。

2.前端配置

前端跨域设置 withCredentials: true

- angular2应用

        const httpOptions = {
          headers: new HttpHeaders({
            'Content-Version': '0',
            'Content-Type':  'application/json'
          }),
          params: new HttpParams(),
          withCredentials: true
        };
    
关键：`withCredentials: true`。 

- ajax请求

在ajax请求里加上xhrFields: {withCredentials: true}, crossDomain: true。
    

即在http请求中，options参数中添加参数`withCredentials: true`。

这样配置后，经过测试，发现，每次通过zuul路由到具体服务后，在具体服务中的sessionId都是一样的。只创建一次会话，一直保持，知道会话断开。

3.补充（配置）

zuul配置：

    zuul:
      add-host-header: true #重定向问题
      sensitive-headers: #保留所有头信息传递，解决多个服务在转发中sessionId不一致的问题，到了服务层，缺少头信息的问题。注意：每个具体的服务的sessionId还是不一样的?
      ignored-headers: Access-Control-Allow-Origin,H-APP-Id,Token,APPToken         
      
具体服务跨域配置：

    # cors跨域设置
    custom:
      cors:
        mapping: /**
        allowCredentials: false # 不循序cookie跨域
        allowedOrigins: "*" #允许的域，多个用逗号隔开。*允许全部的域通过。一定要用双引号，否则配置文件报错。
        #    allowedMethods: POST,GET,DELETE,PUT #这样设置，静态资源将不能跨域
        allowedMethods: "*"
        allowedHeaders: "*"        

-------------------------------
### feign服务调用问题

#### 问题一

_1.问题描述：_

在服务A中调用服务B，无法调用B的接口，报如下异常：
```text
feign.RetryableException: too many bytes written
```

_2.问题原因：_

是因为feign请求body和Content-Length长度不一致导致。

_3.解决方案：_

在自定义feign拦截器中，过滤掉头信息`content-length`：

```text
if (name.equals("content-length")){
    continue;
}
```
具体如下：

```java
package com.xrlj.framework.config;

import feign.Logger;
import feign.RequestInterceptor;
import feign.RequestTemplate;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

import javax.servlet.http.HttpServletRequest;
import java.util.Enumeration;

/**
 * 放在扫描包下。全局有效。所有feign客户端有效。
 * 配合自定义熔断策略。
 */
@Configuration
public class FeignConfiguration {

    /**
     * 日志级别。FULL打印请求头，请求体等信息。
     * @return
     */
    @Bean
    Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL;
    }

    /**
     * 创建Feign请求拦截器，在发送请求前设置认证的token,各个微服务将token设置到环境变量中来达到通用
     * @return
     * */
    @Bean
    public FeignBasicAuthRequestInterceptor basicAuthRequestInterceptor() {
        return new FeignBasicAuthRequestInterceptor();
    }

    /**
     * Feign请求拦截器
     * @author yinjihuan
     * @create 2017-11-10 17:25
     **/
    public class FeignBasicAuthRequestInterceptor  implements RequestInterceptor {

        public FeignBasicAuthRequestInterceptor() {

        }

        @Override
        public void apply(RequestTemplate template) {
            //配置自定义熔断策略或者在.yml中配置熔断策略为hystrix.command.default.execution.isolation.strategy: SEMAPHORE
            //否则这里返回null
            ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
            if (attributes == null) {
                return;
            }
            HttpServletRequest request = attributes.getRequest();

            //添加所有头信息。feign调用，在各组件中传递。保持各组件之间session一致性，同一个sessionId。
            Enumeration<String> headerNames = request.getHeaderNames();
            if (headerNames != null) {
                while (headerNames.hasMoreElements()) {
                    String name = headerNames.nextElement();
                    Enumeration<String> values = request.getHeaders(name);
                    while (values.hasMoreElements()) {
                        String value = values.nextElement();
                        // 跳过 content-length
                        // https://blog.csdn.net/qq_39986681/article/details/107138740
                        // https://juejin.cn/post/6844903939079421966
                        if (name.equals("content-length")){
                            continue;
                        }
                        template.header(name, value);
                    }
                }
            }
        }
    }
}

```

4.参考：
https://blog.csdn.net/qq_39986681/article/details/107138740
https://juejin.cn/post/6844903939079421966






