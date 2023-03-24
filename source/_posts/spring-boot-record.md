---
title: spring-boot使用记录
date: 2019-05-04 01:09:30
categories: spring-boot
tags:
---

收集spring boot使用过程中的常用点……

## 常见注解

### 注解@ConditionalOnMissingBean

- 作用：标识是否要初始化该Bean。

例子： 

    @Bean
    @ConditionalOnMissingBean
    public DataSourceConnectionProvider dataSourceConnectionProvider(
            DataSource dataSource) {
        return new DataSourceConnectionProvider(
                new TransactionAwareDataSourceProxy(dataSource));
    }

上面初始化一个Bean，但是加了注解@ConditionalOnMissingBean。意思是：如果已经有初始化DataSourceConnectionProvider的Bean，该方法将不会执行。如：

    @Bean
    public DataSourceConnectionProvider connectionProvider() {
        return new DataSourceConnectionProvider(new TransactionAwareDataSourceProxy(DataSourceSpyUtils.conversion(environment,dynamicDataSource)));
    }

上面这段代码，初始化了该Bean，所以上面的将不会再执行。

- 用处：常用在spring boot starter中。

### @ConditionalOnxxx相关注解总结

参考：     
https://www.cnblogs.com/yixianyixian/p/7346894.html     
https://blog.csdn.net/xcy1193068639/article/details/81491071

### 自定义Conditional

https://blog.csdn.net/zhanglu1236789/article/details/78999496

## 集成hikari数据源

### 错误一，maxLifetime问题

    The last packet successfully received from the server was 1,057,018 milliseconds ago.  The last packet sent successfully to the server was 1,057,026 milliseconds ago.). Possibly consider using a shorter maxLifetime value.

该警告代码在：com.zaxxer.hikari.pool.PoolBase类中，可以debbuger进去看。 

解决：

## Redis使用错误

- 错误描述：


    Spring Data Redis - Could not safely identify store assignment for repositor
    
- 原因分析：

1.使用了Spring data jpa 作为持久层框架    
2.使用了Spring Redis 缓存

这是 Spring Boot 的 Autoconfigure 包干的好事，里面有个叫 RedisRepositoriesAutoConfiguration 的类会检查当前的 classpath 里面是不是存在 Jedis 和 @EnableRedisRepositories，如果存在，无论你的代码有没有用，他都会帮你自动启用这个注解（不带参数），于是整个 classpath 的类都会被扫进去。

- 解决：

解决方法也很简单，RedisRepositoriesAutoConfiguration 里面会判断 spring.data.redis.repositories.enable 这个配置项是否存在，不存在、存在和值为 true 都会生效，只要显式设置它为 false 即可；如果不想写配置信息，也不需要用 RedisRepository 的话（不影响 RedisTemplate），可以通过另外一个判断条件——检查 RedisRepositoryFactoryBean 这个 Bean 是否存在来处理，默认是不存在则执行这个 AutoConfiguration，只要自己在代码里造一个 RedisRepositoryFactoryBean 即可，比如这样

    @Bean
    public RedisRepositoryFactoryBean redisRepositoryFactoryBean() {
        return null;
    }
    
也可以直接禁用redis的repositories

    spring.data.redis.repositories.enabled = false    

## 父类中注入泛型bean，爆错多个bean实例无法注入

- 错误描述：

```text
Field dao in com.hgbio.core.base.BaseServiceImpl required a single bean, but 23 were found:
```

- 解决方法：

`给其中一个实现类添加@Primary标记为默认初始化的类即可`

## 重定向路径后，前端出现跨域问题

- 问题描述：

前后端分离的项目，前端angular，后端spring boot（单体应用）。前后端都已经做了跨域的相关配置。在后端定义了过滤器，来拦截请求检测token的时候，如果token错误，那么就重定向到一个错误的路径，此时，出现了跨域的问题。

代码：

```java
 @Override
public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
    log.info(">>>>indexFilter doFilter token");

    HttpServletRequest request = (HttpServletRequest) servletRequest;
    HttpServletResponse response = (HttpServletResponse) servletResponse;
    String path = request.getRequestURI().substring(request.getContextPath().length()).replaceAll("[/]+$", "");
    boolean allowedPath = ALLOWED_PATHS.contains(path);
    if (allowedPath) { //允许直接通过网址
        filterChain.doFilter(servletRequest, servletResponse);
        return;
    }

    //option预检查，直接通过请求
    if ("OPTIONS".equals(request.getMethod())) {
        response.setStatus(HttpServletResponse.SC_OK);
        //filterChain.doFilter(servletRequest, servletResponse);
        return;
    }

    String token = getToken(request);
    log.info(">>>>indexFilter doFilter token: {}", token);
    if (StringUtil.isEmpty(token)) {
        response.sendRedirect("/token/error");
        //request.getRequestDispatcher("/token/error").forward(request, response);
        return;
    }

    filterChain.doFilter(servletRequest, servletResponse);

}
```

- 错误内容：

前端报错：

```text
Access to XMLHttpRequest at 'http://127.0.0.1:9010/user/getUserMenus' from origin 'http://localhost:12000' has been blocked by CORS policy: Response to preflight request doesn't pass access control check: No 'Access-Control-Allow-Origin' header is present on the requested resource.
```

- 解决：

之所以会出现跨域问题，是因为使用`response.sendRedirect("/token/error");`重定向，浏览器默认跳转页面，所以出现跨域问题。因此要改成：`request.getRequestDispatcher("/token/error").forward(request, response);`,该方法重定向只涉及到后端的转发，不涉及到前端，所以不会出现跨域问题。
