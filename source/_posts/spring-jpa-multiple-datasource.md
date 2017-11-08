---
title: spring-data-jpa动态数据源读写分离
date: 2017-11-08 23:02:49
categories: spring-boot
tags: jpa读写分离配置
---

在代码层面配置多数据源，手动或者注解方式自动切换数据源，达到读写分离的目的。可以jpa，jdbc，mybatis共存。

## 1. 配置数据源

采用阿里druid数据源配置连接池。

具体配置如下：

    package service.basic.user.config.ds;
    
    import com.alibaba.druid.filter.Filter;
    import com.alibaba.druid.filter.logging.Log4j2Filter;
    import com.alibaba.druid.filter.stat.StatFilter;
    import com.alibaba.druid.pool.DruidDataSource;
    import com.alibaba.druid.wall.WallConfig;
    import com.alibaba.druid.wall.WallFilter;
    import com.ymu.spcselling.infrastructure.dao.ds.DynamicDataSource;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.beans.factory.annotation.Qualifier;
    import org.springframework.context.annotation.*;
    import org.springframework.core.env.Environment;
    
    import javax.sql.DataSource;
    import java.sql.SQLException;
    import java.util.ArrayList;
    import java.util.HashMap;
    import java.util.List;
    import java.util.Map;
    
    
    /**
     * 配置数据源
     */
    @Configuration
    public class DataSourceConfig {
    
        /**
         * druid监控filter配置。
         * @return
         */
        @Bean
        public StatFilter statFilter() {
            StatFilter statFilter = new StatFilter();
            statFilter.setSlowSqlMillis(5 * 1000); //超过5秒执行的为慢sql
            statFilter.setLogSlowSql(true); //日志记录慢sql
            statFilter.setMergeSql(true); //相同sql合并
            return statFilter;
        }
    
        //----------- sql注入攻击防御配置 start ---------//
    
        @Bean
        public WallConfig wallConfig() {
            WallConfig wallConfig = new WallConfig();
            wallConfig.setDir("classpath:druid/wall/mysql"); //sql过滤规则装载位置。
            return wallConfig;
        }
    
        @Bean
        public WallFilter wallFilter() {
            WallFilter wallFilter = new WallFilter();
            wallFilter.setDbType("mysql"); //指定数据库类型。
            wallFilter.setConfig(wallConfig());
            return wallFilter;
        }
    
        //----------- sql注入攻击防御配置 end ---------//
    
    
        /**
         * 打印sql语句。
         * @return
         */
        @Bean(name = "log4j2Filter")
        public Log4j2Filter log4j2Filter() {
            Log4j2Filter log4j2Filter = new Log4j2Filter();
            log4j2Filter.setConnectionLogEnabled(false);
            log4j2Filter.setResultSetLogEnabled(true); //显示sql
            log4j2Filter.setDataSourceLogEnabled(false);
            log4j2Filter.setStatementExecutableSqlLogEnable(true); //输出可执行的SQL
            log4j2Filter.setStatementLogEnabled(false);
            return log4j2Filter;
        }
    
    
    
        //-------------- 数据源配置 start ---------------//
    
        /**
         * 会员主库（spcs_user）数据源。
         *
         * @return
         * @throws SQLException
         */
        @Bean(name = "spcsUserDataSourceWrite")
        @Qualifier("spcsUserDataSourceWrite")
        public DataSource spcsUserDataSource(@Autowired SpcsUserDSArgs args) throws SQLException {
            DruidDataSource dataSource = new DruidDataSource();
            dataSource.setUrl(args.getUrl());
            dataSource.setUsername(args.getUsername());
            dataSource.setPassword(args.getPassword());
            dataSource.setDriverClassName(args.getDriverClassName());
            dataSource.setInitialSize(args.getInitialSize());
            dataSource.setMinIdle(args.getMinIdle());
            dataSource.setMaxActive(args.getMaxActive());
            dataSource.setMaxWait(args.getMaxWait());
            dataSource.setTimeBetweenEvictionRunsMillis(args.getTimeBetweenEvictionRunsMillis());
            dataSource.setMinEvictableIdleTimeMillis(args.getMinEvictableIdleTimeMillis());
    
            dataSource.setUseGlobalDataSourceStat(true); //合并多个DruidDataSource的监控数据
    
            List<Filter> proxyFilters = new ArrayList<>();
            proxyFilters.add(statFilter());
            proxyFilters.add(log4j2Filter());
            proxyFilters.add(wallFilter());
            dataSource.setProxyFilters(proxyFilters);
    
            return dataSource;
        }
    
        /**
         * 会员从库（spcs_user_slave）数据源。
         *
         * @return
         * @throws SQLException
         */
        @Bean(name = "spcsUserDataSourceRead_0")
        @Qualifier("spcsUserDataSourceRead_0")
        public DataSource spcsUserSlaveDataSource(@Autowired SpcsUserSlaveDSArgs args) throws SQLException {
            DruidDataSource dataSource = new DruidDataSource();
            dataSource.setUrl(args.getUrl());
            dataSource.setUsername(args.getUsername());
            dataSource.setPassword(args.getPassword());
            dataSource.setDriverClassName(args.getDriverClassName());
            dataSource.setMinIdle(args.getMinIdle());
            dataSource.setInitialSize(args.getInitialSize());
            dataSource.setMaxActive(args.getMaxActive());
            dataSource.setMaxWait(args.getMaxWait());
            dataSource.setTimeBetweenEvictionRunsMillis(args.getTimeBetweenEvictionRunsMillis());
            dataSource.setMinEvictableIdleTimeMillis(args.getMinEvictableIdleTimeMillis());
    
            List<Filter> proxyFilters = new ArrayList<>();
            proxyFilters.add(statFilter());
            proxyFilters.add(log4j2Filter());
            proxyFilters.add(wallFilter());
            dataSource.setProxyFilters(proxyFilters);
    
            return dataSource;
        }
    
    
        /**
         * 动态数据源: 通过AOP在不同数据源之间动态切换
         *
         * @return
         */
        @Primary
        @Bean(name = "dataSource")
        @Scope("singleton")
        @DependsOn({"spcsUserDataSourceWrite","spcsUserDataSourceRead_0"}) //要加入这个注解，在数据源初始化之后，再初始化本bean，否则会出现循环依赖注入无法启动。
        public DataSource dynamicDataSource(@Qualifier("spcsUserDataSourceWrite") DataSource spcsUserDataSource,
                                              @Qualifier("spcsUserDataSourceRead_0") DataSource spcsUserSlaveDataSource) {
            // 配置多数据源
            Map<Object, Object> dsMap = new HashMap<>(5);
            dsMap.put(DSType.SPCS_USER.name(), spcsUserDataSource);
            dsMap.put(DSType.SPCS_USER_SLAVE.name(), spcsUserSlaveDataSource);
    
            DynamicDataSource dynamicDataSource = new DynamicDataSource();
            // 默认数据源
            dynamicDataSource.setDefaultTargetDataSource(spcsUserDataSource);
            dynamicDataSource.setTargetDataSources(dsMap);
            return dynamicDataSource;
        }
    
    }

> *注意*：dynamicDataSource中一定要加入注解 @Primary，单多个数据元时候，默认取该个，避免无法区分。另外特别注意注解：@DependsOn。一定要加该注解，在实际实际数据源注入后，再注入动态数据源，否则会出现循环依赖导致系统无法启动的局面。

## 2. 实现自己的数据源路由（关键）

相当于多数据源的路由功能。

    package com.ymu.spcselling.infrastructure.dao.ds;
    
    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
    import org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource;
    
    public class DynamicDataSource extends AbstractRoutingDataSource {
    
        private static final Logger log = LoggerFactory.getLogger(DynamicDataSource.class);
    
        @Override
        protected Object determineCurrentLookupKey() {
            log.debug("数据源为{}", DataSourceContextHolder.getDS());
            //可以做一个简单的负载均衡策略
            return DataSourceContextHolder.getDS();
        }
    
    }
    
## 3. 持有数据源

    package com.ymu.spcselling.infrastructure.dao.ds;
    
    
    import org.apache.logging.log4j.LogManager;
    import org.apache.logging.log4j.Logger;
    
    public class DataSourceContextHolder {
    
        private static final Logger LOGGER = LogManager.getLogger(DataSourceContextHolder.class);
    
        private static final ThreadLocal<String> contextHolder = new ThreadLocal<>();
    
        /**
         * 设置数据源名。
         * @param dbType
         */
        public static void setDS(String dbType) {
            if (dbType == null) {
                throw new NullPointerException("数据源不能null");
            }
            LOGGER.debug("切换到{}数据源", dbType);
            contextHolder.set(dbType);
        }
    
        /**
         * 获取数据源名。
         * @return
         */
        public static String getDS() {
            return (contextHolder.get());
        }
    
        /**
         * 清除数据源名。
         */
        public static void clearDS() {
            contextHolder.remove();
        }
    }
    
## 4. 通过aop，注解方式持有数据源

    package service.basic.user.config.ds;
    
    import com.ymu.spcselling.infrastructure.dao.ds.DSInject;
    import com.ymu.spcselling.infrastructure.dao.ds.DataSourceContextHolder;
    import org.aspectj.lang.JoinPoint;
    import org.aspectj.lang.annotation.After;
    import org.aspectj.lang.annotation.Aspect;
    import org.aspectj.lang.annotation.Before;
    import org.aspectj.lang.reflect.MethodSignature;
    import org.springframework.stereotype.Component;
    
    import java.lang.reflect.Method;
    
    /**
     *  解析注入的数据源。
     */
    @Aspect
    @Component
    public class DynamicDataSourceAspect {
    
    	@Before("@annotation(com.ymu.spcselling.infrastructure.dao.ds.DSInject)")
    	public void beforeSwitchDS(JoinPoint point) {
    
    		// 获得当前访问的class
    		Class<?> className = point.getTarget().getClass();
    
    		// 获得访问的方法名
    		String methodName = point.getSignature().getName();
    		// 得到方法的参数的类型
    		Class[] argClass = ((MethodSignature) point.getSignature()).getParameterTypes();
    		String dataSource = DSType.SPCS_USER.name(); //默认主库
    		try {
    			// 得到访问的方法对象
    			Method method = className.getMethod(methodName, argClass);
    
    			// 判断是否存在@DBInject注解
    			if (method.isAnnotationPresent(DSInject.class)) {
    				DSInject annotation = method.getAnnotation(DSInject.class);
    				// 取出注解中的数据源名
    				dataSource = annotation.value();
    			}
    		} catch (Exception e) {
    			e.printStackTrace();
    		}
    
    		// 切换数据源
    		DataSourceContextHolder.setDS(dataSource);
    
    	}
    
    	@After("@annotation(com.ymu.spcselling.infrastructure.dao.ds.DSInject)")
    	public void afterSwitchDS(JoinPoint point) {
    		DataSourceContextHolder.clearDS();
    	}
    }

---
    package com.ymu.spcselling.infrastructure.dao.ds;
    
    import java.lang.annotation.Documented;
    import java.lang.annotation.ElementType;
    import java.lang.annotation.Retention;
    import java.lang.annotation.RetentionPolicy;
    import java.lang.annotation.Target;
    
    @Target({ ElementType.PARAMETER, ElementType.METHOD })
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    public @interface DSInject {
    	String value() default "";
    }

## 5. 配置jpa

    package service.basic.user.config.ds;
    
    import com.ymu.spcselling.infrastructure.dao.BaseRepositoryFactoryBean;
    import net.sf.log4jdbc.sql.jdbcapi.DataSourceSpy;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.beans.factory.annotation.Qualifier;
    import org.springframework.boot.autoconfigure.orm.jpa.JpaProperties;
    import org.springframework.boot.orm.jpa.EntityManagerFactoryBuilder;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.context.annotation.Primary;
    import org.springframework.core.env.Environment;
    import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
    import org.springframework.orm.jpa.JpaTransactionManager;
    import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
    import org.springframework.transaction.PlatformTransactionManager;
    import org.springframework.transaction.annotation.EnableTransactionManagement;
    import service.basic.user.common.Constants;
    
    import javax.persistence.EntityManager;
    import javax.sql.DataSource;
    import java.util.Map;
    
    @Configuration
    @EnableTransactionManagement
    @EnableJpaRepositories(entityManagerFactoryRef = "entityManagerFactorySpcsUserDB", transactionManagerRef = "transactionManagerSpcsUserDB", basePackages = {
            Constants.SPCS_USER_REPOSITORY_PACKAGE_PATH}, repositoryFactoryBeanClass = BaseRepositoryFactoryBean.class)
    public class SpcsUserDBConfig {
    
        @Autowired
        Environment ev;
    
        @Autowired
        @Qualifier("dataSource")
        private DataSource dataSource; // 数据源
    
        @Primary
        @Bean(name = "entityManagerSpcsUser")
        public EntityManager entityManager(EntityManagerFactoryBuilder builder) {
            return entityManagerFactorySpcsUserDB(builder).getObject().createEntityManager();
        }
    
        @Primary
        @Bean(name = "entityManagerFactorySpcsUserDB")
        public LocalContainerEntityManagerFactoryBean entityManagerFactorySpcsUserDB(EntityManagerFactoryBuilder builder) {
            if (ev.acceptsProfiles("dev") || ev.acceptsProfiles("test")
                    || ev.acceptsProfiles("update")) {
                dataSource = new DataSourceSpy(dataSource); // log4jdbc打印sql日志。
            }
            return builder.dataSource(dataSource).properties(getVendorProperties(dataSource))
                    .packages(Constants.SPCS_USER_ENTITY_PACKAGE_PATH)
                    .persistenceUnit("spcsUserUnit").build(); //实体管理器别名,多数据元要设置。
        }
    
        private Map<String, String> getVendorProperties(DataSource dataSource) {
            JpaProperties jpaProperties = new JpaProperties();
            return jpaProperties.getHibernateProperties(dataSource);
        }
    
        /**
         * 开启事务。
         *
         * @param builder
         * @return
         */
        @Primary
        @Bean(name = "transactionManagerSpcsUserDB")
        public PlatformTransactionManager transactionManagerSpcsUserDB(EntityManagerFactoryBuilder builder) {
            return new JpaTransactionManager(entityManagerFactorySpcsUserDB(builder).getObject());
        }
    
    }
    
> *注意*: 注入的数据源为上面配置的动态数据源。
@Autowired
@Qualifier("dataSource")
private DataSource dataSource; // 数据源    


## 6. 配置spring jdbcTemplate

    package service.basic.user.config.ds;
    
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.beans.factory.annotation.Qualifier;
    import org.springframework.boot.autoconfigure.AutoConfigureAfter;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.jdbc.core.JdbcTemplate;
    
    import javax.sql.DataSource;
    
    @Configuration
    @AutoConfigureAfter(DataSourceConfig.class)
    public class JdbcTemplateConfig {
    
        @Autowired
        @Qualifier(value = "dataSource")
        private DataSource dataSource;
    
        /**
         * spring jdbc。
         *
         * @return
         */
        @Bean(name = "jdbcTemplate")
        @Qualifier("jdbcTemplate")
        public JdbcTemplate jdbcTemplate() {
            return new JdbcTemplate(dataSource);
        }
    }


## 7. 使用

- 方式一：
可在service层，也可在dao层做。
在开始操作数据库前调用：


    DataSourceContextHolder.setDS(DSType.SPCS_USER_SLAVE.name());
    
    //查询数据
    
    DataSourceContextHolder.clearDS();
    
- 方式二：
通过注解，可在service层，也可在dao层做。


    @DSInject(value = Constants.SPCS_USER_SLAVE)
    @Override
    public User getUserByMobile(String mobile) {
        return userDao.findUserByMobile(mobile);
    }    