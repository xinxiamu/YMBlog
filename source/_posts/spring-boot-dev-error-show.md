---
title: spring-boot开发常见异常收录
date: 2017-09-29 00:31:25
categories: spring-boot
tags: 
---

## jpa篇

### 1 异常一：缺少jpa数据源配置
异常描述：Cannot determine embedded database driver class for database type NONE

原因：该异常在spring-boot应用启动时候报异常。是因为maven依赖中依赖如了jpa，所以系统会自动配置试图注入jpa数据源。但是如果没又配置数据源，则会报该异常。

#### 1.1 处理方法一
在pom中剔除jpa注入

     <dependency>
         <groupId>com.ymu.spcselling</groupId>
         <artifactId>spcselling-infrastructure</artifactId>
         <exclusions>
             <exclusion>
                 <groupId>org.springframework.boot</groupId>
                 <artifactId>spring-boot-starter-data-jpa</artifactId>
             </exclusion>
         </exclusions>
     </dependency>
     
#### 1.2 不传递依赖

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
        <optional>true</optional><!--不在子应用中传递-->
    </dependency>    

    
#### 1.3 在@SpringBootApplication中排除其注入

    @SpringBootApplication(exclude={DataSourceAutoConfiguration.class,HibernateJpaAutoConfiguration.class})

### 2 自动创建表指定Mysql搜索引擎类型

解决方法，只需要在配置文件添加如下代码:
    
    # 指定生成表名的存储引擎为InneoDB
    spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect

## 启动类配置

1.让一个类型不被ComponentScan扫描

    @ComponentScan(excludeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION,
            value = {ExcludeComponent.class})) //添加了@ExcludeComponent注解的类将不会被ComponentScan扫描
    public class ServiceFileclientApplication {
    
        public static void main(String[] args) {
            SpringApplication.run(ServiceFileclientApplication.class, args);
        }
    }
    
    ----
    
    package com.ymu.servicefileclient.config;
    
    public @interface ExcludeComponent {
    }
    
    -------
    /**
     * 该类为Feign的配置类
     * 注意：该类不应该在主应用程序上下文的@CompantScan中
     */
    @ExcludeComponent
    @Configuration
    public class FeignConfiguration {
    
        /**
         * 用feign.Contract.Default替换SpringMvcContract契约
         *
         * @return
         */
        @Bean
        public Contract feignContract() {
            return new feign.Contract.Default();
        }
    
    }

## 文件相关

1.在jar包所在目录下创建文件或者文件夹

     //运行jar所在文件目录
     @Value("${user.dir}")
     private String userDir;
     
     //创建文件夹abc
      URI uri = URI.create(userDir);
      File file = new File(uri + "/abc");
      if (!file.exists()) {
          file.mkdir();
      }

2.系统用户根目录下创建文件

     // 桌面路径
    FileSystemView fsv = FileSystemView.getFileSystemView();
    File com = fsv.getHomeDirectory();
    // String url = com.getPath().replaceAll("\\\\", "\\\\\\\\") + "\\";
    String filePath = com.getPath();
    String fileName = "导出数据.pdf";
    String uri = filePath.concat(File.separator).concat(fileName);   
   
3.获取类所在资源目录

    this.getClass().getClassLoader().getResource("").getPath(); 
    
    --------------------------------------x
    
    近期在用springboot封装一些对外服务的API接口，在本机测试都很顺利，可是当我打包jar文件放到服务器上测试的时候发现了类似下面的异常信息：
    
    java.nio.file.NoSuchFileException: file:/app.jar!/BOOT-INF/classes!/xxx.properties 
     
    于是网上一番搜索，找到类似的解决方法：
    Properties prop = new Properties();
    InputStream is = this.getClass().getResourceAsStream(filePath);     
    
## 取消自动配置

### 方式一

去掉pom中相关的依赖包。

### 代码中配置

1.使用了@EnableAutoConfiguration的时候    

    @EnableAutoConfiguration(exclude={DataSourceAutoConfiguration.class})
    
2.使用了@SpringBootApplication的时候

    @SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
    1
3.使用了@SpringCloudApplication的时候

    @EnableAutoConfiguration(exclude={DataSourceAutoConfiguration.class})
    @SpringCloudApplication
    
4.通过配置文件来设置

    spring.autoconfigure.exclude=org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration
    1
    
  
## spring boot tomcat临时文件夹的问题
 
###  The temporary upload location [/tmp/tomcat.1337767218595042057.80/work/Tomcat/localhost/ROOT] is no 

线上的系统中不能上传文件了，出现如下错误：

    org.springframework.web.multipart.MultipartException: Could not parse multipart servlet request;nested exception is java.io.IOException: The temporary upload location [/tmp/tomcat.1337767218595042057.80/work/Tomcat/localhost/ROOT] is not valid 
    
原因： 
在linux系统中，springboot应用服务再启动（java -jar 命令启动服务）的时候，会在操作系统的/tmp目录下生成一个tomcat*的文件目录，上传的文件先要转换成临时文件保存在这个文件夹下面。由于临时/tmp目录下的文件，在长时间（10天）没有使用的情况下，就会被系统机制自动删除掉。所以如果系统长时间无人问津的话，就可能导致上面这个问题。

临时解决方法： 
手动在/tmp下创建相应得文件夹或者重启系统

解决办法： 
在 yml配置文件 中添加：

    server.tomcat.basedir: /data/apps/temp  
    
手动的将临时文件夹设置为自定义的文件夹，就不会被Linux删除了。    
    
 
 ## 集成jooq后，添加aop切面，应用启动变慢的问题
 
- 问题描述：
 
 spring boot2项目中，jpa集成jooq，添加切面，然后启动程序，每次都卡在初始化jpa的地方，要等待一两分钟甚至更久才能启动应用。

aop表达式：

    @Before("execution(public * com.xrlj.servicesyscommon.service.impl..*.*(..)))")
    
启动会卡在这里：    

    2019-04-15 00:21:42.100  INFO 9072 --- [  restartedMain] log4jdbc.log4j2                          : 2. Connection closed. {executed in 0ms} 
    2019-04-15 00:21:42.100  INFO 9072 --- [  restartedMain] log4jdbc.log4j2                          : 2. Connection.close() returned 
    2019-04-15 00:21:42.122  INFO 9072 --- [  restartedMain] j.LocalContainerEntityManagerFactoryBean : Initialized JPA EntityManagerFactory for persistence unit 'myself_db'

- 问题解决：
把execution表达式，改成within表达式即可。    

改成：

    @Before("within(com.xrlj.servicesyscommon.service.impl..*)")

即可。

参考：
网上描述，说是集成了jooq后才会这样的。   
https://github.com/jOOQ/jOOQ/issues/5902
     
    
