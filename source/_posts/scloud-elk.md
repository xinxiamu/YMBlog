---
title: Spring Cloud Sleuth使用ELK收集&分析日志
date: 2019-09-04 22:15:36
categories: spring-cloud
tags: ELK
---

在分布式系统中，每个应用的实例都会在不同的物理机上产生日志。以往，系统出现问题需要查看日志的时候，就要到每台物理机对应的日志目录下查看相应的日志，这会非常的麻烦，特别是很多个实例的时候。      
另外，微服务场景下，各个服务的日志链也都很分散，无法追踪，不知道实际的错误出现在哪个服务。      
所以，就非常有必要把各个服务的各个实例所产生的日志都发送到统一的服务器上，并进行可视化的查看，分析。这就是我们这里要介绍的ELK能做的事情了。

## ELK是什么东西

实际就是三个软件系统：

- E：指的Elasticsearch,一个强大的搜索引擎。
- L：指的Logstash，解析并收集日志的系统。
- K：指的Kibana，一个可以可视化查看日志文件的系统。

## 基本原理

- Sleuth打印JSON格式的日志。
- logstash中配置语法，解析并收集JSON格式的日志，然后存储到Elasticsearch系统中去。
- Kibana可视化分析日志。集成Elasticsearch强大的搜索功能，找到任意输出的日志，并查看和分析。

## ELK环境的搭建

这里介绍采用的单机模式。利用docker和docker-compose来部署。

首先要在某台服务器上安装docker引擎和docker-compose服务。

- 在服务器新建文件夹ELK。然后进入该文件夹，并创建文件`docker-compose.yml`，内容如下：


        version: '3'
        services:
          elasticsearch:
            image: elasticsearch:7.3.1
            environment:
              discovery.type: single-node
            ports:
              - "9200:9200"
              - "9300:9300"
          logstash:
            image: logstash:7.3.1
            command: logstash -f /etc/logstash/conf.d/logstash.conf
            volumes:
              # 挂载logstash配置文件
              - ./config:/etc/logstash/conf.d
              - /opt/build:/opt/build
            ports:
              - "5000:5000"
              - "8088:8088"
          kibana:
            image: kibana:7.3.1
            environment:
              - ELASTICSEARCH_URL=http://120.79.2.30:9200
            ports:
              - "5601:5601"
          
- 在ELK中再创建目录config，并进入config目录，然后创建文件`logstash.conf`,文件内容如下：

    
    input {
        tcp {
            port => 8088
            mode => "server"
            ssl_enable => false
            type => "tcplog"
            codec => json_lines {
                charset => "UTF-8"
            }
        }
    }
    filter {
        grok {
            match => { "message" => "%{TIMESTAMP_ISO8601:timestamp}\s+%{LOGLEVEL:severity}\s+\[%{DATA:service},%{DATA:trace},%{DATA:span},%{DATA:exportable}\]\s+%{DATA:pid}\s+---\s+\[%{DATA:thread}\]\s+%{DATA:class}\s+:\s+%{GREEDYDATA:rest}" }
        }
    }
    output {
        elasticsearch {
            hosts => "120.79.2.30:9200"
            index => "logstash"
        }
    }
    
- 在ELK目下，启动ELK服务：


    docker-compose up 
    
## spring cloud 服务配置调整

### pom.xml文件中添加依赖

    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-sleuth</artifactId>
    </dependency>
    <!--必须要和logback版本对应，这个版本对应logback的1.2.3-->
    <dependency>
        <groupId>net.logstash.logback</groupId>
        <artifactId>logstash-logback-encoder</artifactId>
        <version>6.1</version>
    </dependency> 
    
 _注意_: logstash-logback-encoder 的版本务必和Logback兼容，否则会导致应用启动不起来，而且不会打印任何日志！可前往 `https://github.com/logstash/logstash-logback-encoder` 查看和Logback的兼容性。                


### 添加logback-spring.xml文件

在 resources 目录下创建配置文件：logback-spring.xml，文件内容如下：

    <?xml version="1.0" encoding="UTF-8"?>
    <configuration>
        <include resource="org/springframework/boot/logging/logback/defaults.xml"/>
    
        <springProperty scope="context" name="springAppName" source="spring.application.name"/>
        <!-- Example for logging into the build folder of your project -->
        <property name="LOG_FILE" value="./logs/${springAppName}/${springAppName}"/>
    
        <!-- You can override this to have a custom pattern -->
        <property name="CONSOLE_LOG_PATTERN"
                  value="%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}"/>
    
        <property name="FILE_LOG_PATTERN" value="${FILE_LOG_PATTERN:-%d{${LOG_DATEFORMAT_PATTERN:-yyyy-MM-dd HH:mm:ss.SSS}} ${LOG_LEVEL_PATTERN:-%5p} ${PID:- } --- [%t] %-40.40logger{39} : %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}"/>
    
        <!-- Appender to log to console -->
        <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
            <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
                <!-- Minimum logging level to be presented in the console logs-->
                <level>DEBUG</level>
            </filter>
            <encoder>
                <pattern>${CONSOLE_LOG_PATTERN}</pattern>
                <charset>utf8</charset>
            </encoder>
        </appender>
    
        <!-- Appender to log to file -->
        <appender name="flatfile" class="ch.qos.logback.core.rolling.RollingFileAppender">
            <file>${LOG_FILE}.log</file>
            <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                <fileNamePattern>${LOG_FILE}.%d{yyyy-MM-dd}.gz</fileNamePattern>
                <maxHistory>7</maxHistory>
            </rollingPolicy>
            <encoder>
                <pattern>${FILE_LOG_PATTERN}</pattern>
                <charset>utf8</charset>
            </encoder>
        </appender>
    
        <!--通过网络，把日志发送到ELK服务器-->
        <appender name="logstash" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
            <destination>120.79.2.30:8088</destination>
            <writeBufferSize>16384</writeBufferSize>
            <!-- encoder is required -->
            <encoder class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder">
                <providers>
                    <timestamp>
                        <timeZone>UTC</timeZone>
                    </timestamp>
                    <pattern>
                        <pattern>
                            {
                            "severity": "%level",
                            "service": "${springAppName:-}",
                            "trace": "%X{X-B3-TraceId:-}",
                            "span": "%X{X-B3-SpanId:-}",
                            "parent": "%X{X-B3-ParentSpanId:-}",
                            "exportable": "%X{X-Span-Export:-}",
                            "pid": "${PID:-}",
                            "thread": "%thread",
                            "class": "%logger{40}",
                            "rest": "%message"
                            }
                        </pattern>
                    </pattern>
                </providers>
            </encoder>
        </appender>
    
        <root level="INFO">
            <appender-ref ref="console"/>
            <!-- uncomment this to have also JSON logs -->
            <appender-ref ref="logstash"/>
            <appender-ref ref="flatfile"/>
        </root>
    </configuration>
    
_注意：_ 应用名称`spring.application.name`必须放在配置文件`bootstrap.yml`中，否则`logback-spring.xml`将读取不到该变量。

## 测试Sleuth & ELK

- 启动你的微服务，并访问相关API产生一些输出日志。

- 访问 http://localhost:5601 （Kibana地址），可看到类似如下的界面，按照如图配置Kibana。

{% asset_img kibana1.png %}

按下面图画红框步骤继续配置：

{% asset_img kibana2.png %}

输入查询条件，就可查询日志了：

{% asset_img kibana3.png %}
