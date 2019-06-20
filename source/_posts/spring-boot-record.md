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

