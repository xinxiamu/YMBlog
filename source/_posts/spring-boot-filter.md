---
title: spring-boot过滤器篇
date: 2018-04-26 14:40:54
categories: spring-boot
tags: spring-boot-filter
---

本文记录在spring-boot环境下，添加配置过滤器。以及过滤器的一些常见使用……

## 在spring-boot项目中添加过滤器

### 方式一

1.创建类TestFilter,并实现Filter接口

    package com.ymu.servicecommon.filter;
    
    import org.apache.logging.log4j.LogManager;
    import org.apache.logging.log4j.Logger;
    
    import javax.servlet.*;
    import java.io.IOException;
    
    /**
     * 功能简述:<br>
     *     过滤器配置测试。
     *
     * @author zmt
     * @create 2018-04-26 下午5:15
     * @updateTime
     * @since 1.0.0
     */
    public class TestFilter implements Filter {
    
        protected final Logger logger = LogManager.getLogger(this.getClass());
    
        @Override
        public void init(FilterConfig filterConfig) throws ServletException {
            logger.debug(">>>>testFilter init");
        }
    
        @Override
        public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
            logger.debug(">>>>testFilter doFilter");
            filterChain.doFilter(servletRequest,servletResponse);
        }
    
        @Override
        public void destroy() {
            logger.debug(">>>>testFilter destroy");
        }
    }

2.注解bean

    package com.ymu.servicecommon.config;
    
    import com.ymu.servicecommon.filter.TestFilter;
    import org.springframework.boot.web.servlet.FilterRegistrationBean;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    
    @Configuration
    public class MainConfig {
    
        /**
         * 配置过滤器
         * @return
         */
        @Bean
        public FilterRegistrationBean indexFilterRegistration() {
            FilterRegistrationBean registration = new FilterRegistrationBean(new TestFilter());
            registration.addUrlPatterns("/*");
    //        registration.addInitParameter("abc", "abc-value");
            registration.setName("testFilter");
            return registration;
        }
    
    }


### 方式二

1.创建过滤器IndexFilter2

    package com.example.filter;
    import java.io.IOException;
    import javax.servlet.Filter;
    import javax.servlet.FilterChain;
    import javax.servlet.FilterConfig;
    import javax.servlet.ServletException;
    import javax.servlet.ServletRequest;
    import javax.servlet.ServletResponse;
    import javax.servlet.annotation.WebFilter;
    
    @WebFilter(urlPatterns = "/*", filterName = "indexFilter2")
    public class IndexFilter2 implements Filter{
      @Override
      public void destroy() {
        System.out.println("filter2 destroy method");
      }
      @Override
      public void doFilter(ServletRequest arg0, ServletResponse arg1, FilterChain arg2)
          throws IOException, ServletException {
        System.out.println("filter2 doFilter method");
      }
      @Override
      public void init(FilterConfig arg0) throws ServletException {
        System.out.println("filter2 init method");
      }
    }

2.在程序启动入库加入注解`ServletComponentScan`

    @SpringBootApplication
    @ServletComponentScan
    public class SpringBootSimpleApplication {
      public static void main(String[] args) {
        SpringApplication.run(SpringBootSimpleApplication.class, args);
      }
    }

## 多个过滤器调用顺序

在spring-boot中通过注解`@Order`来标识。这个order的默认值是Integer.MAX_VALUE 也就是int的最大值。多个过滤器会按照order属性的大小从小到大执行。

1.代码设置

    package com.ymu.servicecommon.config;
    
    import com.ymu.servicecommon.filter.Test2Filter;
    import com.ymu.servicecommon.filter.TestFilter;
    import org.springframework.boot.web.servlet.FilterRegistrationBean;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    
    @Configuration
    public class MainConfig {
    
        /**
         * 配置过滤器
         * @return
         */
        @Bean
        public FilterRegistrationBean testFilterRegistration() {
            FilterRegistrationBean registration = new FilterRegistrationBean(new TestFilter());
            registration.addUrlPatterns("/*");
    //        registration.addInitParameter("abc", "abc-value");
            registration.setName("testFilter");
            registration.setOrder(Integer.MAX_VALUE); //设置过滤器执行先后顺序，多个按从小到大执行
            return registration;
        }
    
        @Bean
        public FilterRegistrationBean test2FilterRegistration() {
            FilterRegistrationBean registration = new FilterRegistrationBean(new Test2Filter());
            registration.addUrlPatterns("/*");
    //        registration.addInitParameter("abc", "abc-value");
            registration.setName("test2Filter");
            registration.setOrder(Integer.MAX_VALUE-1); //设置过滤器执行先后顺序，多个按从小到大执行
            return registration;
        }
    
    }
    
----------------------------------------------------    

- 启动程序，观察执行顺序。

{% asset_img a.png %} 

- 请求接口，观察执行顺序。

{% asset_img b.png %}


## 在过滤器中为请求添加http请求头信息

1.定义类ModifyHttpServletRequestWrapper.java

    package com.ymu.framework.web;
    
    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletRequestWrapper;
    import java.util.*;
    
    public class ModifyHttpServletRequestWrapper extends HttpServletRequestWrapper {
     
        private Map<String, String> customHeaders;
     
        public ModifyHttpServletRequestWrapper(HttpServletRequest request) {
            super(request);
            this.customHeaders = new HashMap<>();
        }
     
        public void putHeader(String name, String value) {
            this.customHeaders.put(name, value);
        }
     
        public String getHeader(String name) {
            String value = this.customHeaders.get(name);
            if (value != null) {
                return value;
            }
            return ((HttpServletRequest) getRequest()).getHeader(name);
        }
     
        public Enumeration<String> getHeaderNames() {
            Set<String> set = new HashSet<>(customHeaders.keySet());
            Enumeration<String> enumeration = ((HttpServletRequest) getRequest()).getHeaderNames();
            while (enumeration.hasMoreElements()) {
                String name = enumeration.nextElement();
                set.add(name);
            }
            return Collections.enumeration(set);
        }
     
    }
    
2.定义过滤器，编辑
    
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        logger.debug(">>>>indexFilter doFilter");

        //添加或者更改header信息
        HttpServletRequest request = (HttpServletRequest) servletRequest;
        String apiVersion = request.getHeader("Content-Version");
        if (null == apiVersion || "".equals(apiVersion)) {
            ModifyHttpServletRequestWrapper requestWrapper = new ModifyHttpServletRequestWrapper(request);
            requestWrapper.putHeader("Content-Version","-1");
            filterChain.doFilter(requestWrapper,servletResponse);
        } else {
            filterChain.doFilter(request,servletResponse);
        }
    }   

    重点代码：
    ModifyHttpServletRequestWrapper requestWrapper = new ModifyHttpServletRequestWrapper(request);
    requestWrapper.putHeader("Content-Version","-1");
    filterChain.doFilter(requestWrapper,servletResponse);