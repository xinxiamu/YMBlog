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