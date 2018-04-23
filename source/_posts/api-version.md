---
title: rest-api版本迭代管理实践
date: 2017-09-26 14:23:44
categories: api-manage
tags: api-version-manage,spring-boot中API版本迭代管理
---

在API系统设计，特别是有android或者ios移动客户端的系统设计过程中，当业务发生比较大的变动的时候，就会出现一个问题：我们为了使客户端的新旧版本（客户端有的用户可能不会主动升级版本）能准确的访问api接口并得到准确的数据，我们就不得不在该api接口实现中写代码做兼容。这样的话，随着业务的不断调整，整个api接口实现将变得臃肿不堪，同时bug不断，导致不能适用各个版本客户端的请求。  
因此，对api接口做版本迭代，让接口实现变得简单、易于维护、减少bug就显得十分必要了。  

通常，restful-api的版本迭代实现方式主要又两种：

- 在url中显示设置，如：`https://api.example.com/v1/`。
- 在http请求头中添加，如：
        
        设置请求头：
        Content-Version: 1
        
        请求:
        https://api.example.com
 
-------------------------------------------------------------
 
首先，建立spring-boot-web项目：

{% asset_img a.png %}
 
        
## 在请求头中设置

1.创建注解类ApiVersion
在controller中添加注解标志api版本


    package com.ymu.demo.springboot2apiversion.version;
    
    import org.springframework.web.bind.annotation.Mapping;
    
    import java.lang.annotation.*;
    
    /**
     * 
     * 接口版本标识注解
     *
     */
    @Target({ElementType.METHOD, ElementType.TYPE})
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    @Mapping
    public @interface ApiVersion {
        int value();
    }

2.创建处理类ApiVersionCondition
继承RequestCondition，每次url请求都会首先进入该方法。


    package com.ymu.demo.springboot2apiversion.version;
    
    import org.springframework.web.servlet.mvc.condition.RequestCondition;
    
    import javax.servlet.http.HttpServletRequest;
    import java.util.regex.Pattern;
    
    public class ApiVersionCondition implements RequestCondition<ApiVersionCondition> {
    
        private int apiVersion;
    
        public ApiVersionCondition(int apiVersion){
            this.apiVersion = apiVersion;
        }
    
        public ApiVersionCondition combine(ApiVersionCondition other) {
            // 采用最后定义优先原则，则方法上的定义覆盖类上面的定义
            return new ApiVersionCondition(other.getApiVersion());
        }
    
        public ApiVersionCondition getMatchingCondition(HttpServletRequest request) {
        	String path = request.getServletPath();
        	if (path == null) {
    			return null;
    		}
    		String contentVersion = request.getHeader("Content-Version"); //在http请求头中定义api版本，而不是在url中
            if (null == contentVersion || "".equals(contentVersion)) {
                throw new IllegalArgumentException("Content-Version非null非空");
            }
            if (!isInteger(contentVersion)) {
                throw new IllegalArgumentException("Content-Version必须为整数");
            }
    
            int version = Integer.valueOf(contentVersion).intValue();
            if(version >= this.apiVersion) { // 如果请求的版本号大于配置版本号， 则满足
                return this;
            }
            return null;
        }
    
        public int compareTo(ApiVersionCondition other, HttpServletRequest request) {
            // 优先匹配最新的版本号
            return other.getApiVersion() - this.apiVersion;
        }
    
        public int getApiVersion() {
            return apiVersion;
        }
    
        /**
         * 判断字符串是否为整数。
         * @param str
         * @return
         */
        private boolean isInteger(String str) {
            Pattern pattern = Pattern.compile("^[-\\+]?[\\d]*$");
            return pattern.matcher(str).matches();
        }
    
    }
    
3.自定义url注册回调类CustomRequestMappingHandlerMapping
url注解回调句柄类。继承RequestMappingHandlerMapping。

    package com.ymu.demo.springboot2apiversion.version;
    
    import org.springframework.core.annotation.AnnotationUtils;
    import org.springframework.web.servlet.mvc.condition.RequestCondition;
    import org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping;
    
    import java.lang.reflect.Method;
    
    public class CustomRequestMappingHandlerMapping extends RequestMappingHandlerMapping {
    
        /**
         * 类。
         * @param handlerType
         * @return
         */
        @Override
        protected RequestCondition<ApiVersionCondition> getCustomTypeCondition(Class<?> handlerType) {
            ApiVersion apiVersion = AnnotationUtils.findAnnotation(handlerType, ApiVersion.class); 
            return createCondition(apiVersion);
        }
    
        /**
         * 方法
         * @param method
         * @return
         */
        @Override
        protected RequestCondition<ApiVersionCondition> getCustomMethodCondition(Method method) {
            ApiVersion apiVersion = AnnotationUtils.findAnnotation(method, ApiVersion.class);
            return createCondition(apiVersion);
        }
        
        private RequestCondition<ApiVersionCondition> createCondition(ApiVersion apiVersion) {
            return apiVersion == null ? null : new ApiVersionCondition(apiVersion.value());
        }
    }
    

4.创建web配置类并编辑内容：WebConfig
配置自定义类RequestMappingHandlerMapping。
    
    package com.ymu.demo.springboot2apiversion;
    
    import com.ymu.demo.springboot2apiversion.version.CustomRequestMappingHandlerMapping;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.web.servlet.config.annotation.WebMvcConfigurationSupport;
    import org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping;
    
    @Configuration
    public class WebConfig extends WebMvcConfigurationSupport{
    
        @Override
        @Bean
        public RequestMappingHandlerMapping requestMappingHandlerMapping() {
            RequestMappingHandlerMapping handlerMapping = new CustomRequestMappingHandlerMapping();
            handlerMapping.setOrder(0);
            handlerMapping.setInterceptors(getInterceptors()); 
            return handlerMapping;
        }
        
    }

5.创建演示类HelloController

    package com.ymu.demo.springboot2apiversion;
    
    import com.ymu.demo.springboot2apiversion.version.ApiVersion;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;
    import org.springframework.web.bind.annotation.RestController;
    
    import javax.servlet.http.HttpServletRequest;
    
    @RestController
    @RequestMapping
    public class HelloController {
    	
    	//---------------- api版本管理 demo start ------------------//
    
        @RequestMapping(value = "/hello",method = RequestMethod.GET)
        public String hello0(HttpServletRequest request){
            print(request);
            return "hello";
        }
    
    	@RequestMapping(value = "/hello",method = RequestMethod.GET)
        @ApiVersion(1)
        public String hello1(HttpServletRequest request){
            print(request);
            return "hello:v1";
        }
    
    
        @RequestMapping(value = "/hello",method = RequestMethod.GET)
        @ApiVersion(5)
        public String hello5(HttpServletRequest request){
            print(request);
            return "hello:v5";
        }
    
        @RequestMapping(value = "/hello",method = RequestMethod.GET)
        @ApiVersion(2)
        public String hello2(HttpServletRequest request){
            print(request);
            return "hello:v2";
        }
    
        private void print(HttpServletRequest request) {
            System.out.println("version:" + request.getHeader("Content-Version"));
        }
        
    }    

6.演示：

{% asset_img b.png %}

------------------------------

{% asset_img c.png %}
    

## 在url中显示设置

在url中显示设置基本和上面过程一样。只不过是要稍微调整下注册路径，在注册路径中添加版本号。 

1.修改CustomRequestMappingHandlerMapping类。
只需要重写方法：

    /**
     * 为所有注册路径添加"/{version}"匹配规则。目的，做api版本管理。
     * 不用在每个类或方法的@RequestMapping中加。
     * @param method
     * @param handlerType
     * @return
     */
    @Override
    protected RequestMappingInfo getMappingForMethod(Method method, Class<?> handlerType) {
        RequestMappingInfo requestMappingInfo = super.getMappingForMethod(method, handlerType);
        if (requestMappingInfo != null) {
            PatternsRequestCondition pcOri = requestMappingInfo.getPatternsCondition();
            Set<String> s = pcOri.getPatterns();
            StringBuilder pathNew = new StringBuilder("");
            if (s != null && !s.isEmpty()) {
                for (String str: s ) {
                    if (!"/error".equals(str)) {
                        pathNew.append("/{version}");
                        pathNew.append(str);
                    } else {
                        pathNew.append(str);
                    }
                }
            }

            PatternsRequestCondition pcnNew = new PatternsRequestCondition(pathNew.toString());

            RequestMappingInfo requestMappingInfoNew = new RequestMappingInfo(requestMappingInfo.getName(),pcnNew,requestMappingInfo.getMethodsCondition(),requestMappingInfo.getParamsCondition(),requestMappingInfo.getHeadersCondition(),requestMappingInfo.getConsumesCondition(),requestMappingInfo.getProducesCondition(),requestMappingInfo.getCustomCondition());

            return requestMappingInfoNew;

        }
        return requestMappingInfo;
    }
        
    -----------------------------------------------------
        
    完整代码如下：    

    package com.ymu.framework.spring.mvc.api;
    
    import org.springframework.core.annotation.AnnotationUtils;
    import org.springframework.web.servlet.mvc.condition.PatternsRequestCondition;
    import org.springframework.web.servlet.mvc.condition.RequestCondition;
    import org.springframework.web.servlet.mvc.method.RequestMappingInfo;
    import org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping;
    
    import java.lang.reflect.Method;
    import java.util.Set;
    
    public class CustomRequestMappingHandlerMapping extends RequestMappingHandlerMapping {
    
        /**
         * 为所有注册路径添加"/{version}"匹配规则。目的，做api版本管理。
         * 不用在每个类或方法的@RequestMapping中加。
         * @param method
         * @param handlerType
         * @return
         */
        @Override
        protected RequestMappingInfo getMappingForMethod(Method method, Class<?> handlerType) {
            RequestMappingInfo requestMappingInfo = super.getMappingForMethod(method, handlerType);
            if (requestMappingInfo != null) {
                PatternsRequestCondition pcOri = requestMappingInfo.getPatternsCondition();
                Set<String> s = pcOri.getPatterns();
                StringBuilder pathNew = new StringBuilder("");
                if (s != null && !s.isEmpty()) {
                    for (String str: s ) {
                        if (!"/error".equals(str)) {
                            pathNew.append("/{version}");
                            pathNew.append(str);
                        } else {
                            pathNew.append(str);
                        }
                    }
                }
    
                PatternsRequestCondition pcnNew = new PatternsRequestCondition(pathNew.toString());
    
                RequestMappingInfo requestMappingInfoNew = new RequestMappingInfo(requestMappingInfo.getName(),pcnNew,requestMappingInfo.getMethodsCondition(),requestMappingInfo.getParamsCondition(),requestMappingInfo.getHeadersCondition(),requestMappingInfo.getConsumesCondition(),requestMappingInfo.getProducesCondition(),requestMappingInfo.getCustomCondition());
    
                return requestMappingInfoNew;
    
            }
            return requestMappingInfo;
        }
    
        @Override
        protected RequestCondition<ApiVersionCondition> getCustomTypeCondition(Class<?> handlerType) {
            ApiVersion apiVersion = AnnotationUtils.findAnnotation(handlerType, ApiVersion.class); 
            return createCondition(apiVersion);
        }
    
        @Override
        protected RequestCondition<ApiVersionCondition> getCustomMethodCondition(Method method) {
            ApiVersion apiVersion = AnnotationUtils.findAnnotation(method, ApiVersion.class);
            return createCondition(apiVersion);
        }
        
        private RequestCondition<ApiVersionCondition> createCondition(ApiVersion apiVersion) {
            return apiVersion == null ? null : new ApiVersionCondition(apiVersion.value());
        }
    }

2.修改类ApiVersionCondition。
主要修改方法：getMatchingCondition

    public ApiVersionCondition getMatchingCondition(HttpServletRequest request) {
    //    	String pathInfo = request.getPathInfo();//这个方法获取是null，报错。
        String path = request.getServletPath(); 
        if (path == null) {
            return null;
        }
        Matcher m = VERSION_PREFIX_PATTERN.matcher(path);//匹配路径
        if(m.find()){
            Integer version = Integer.valueOf(m.group(1));
            if(version >= this.apiVersion) // 如果请求的版本号大于配置版本号， 则满足
                return this;
        }
        return null;
    }
    
    -----------------------------------------------------------
    
    完整代码如下：

    package com.ymu.framework.spring.mvc.api;
    
    import java.util.regex.Matcher;
    import java.util.regex.Pattern;
    
    import javax.servlet.http.HttpServletRequest;
    
    import org.springframework.web.servlet.mvc.condition.RequestCondition;
    
    public class ApiVersionCondition implements RequestCondition<ApiVersionCondition> {
    
        // 路径中版本的前缀， 这里用 /v[1-9]/的形式
        private final static Pattern VERSION_PREFIX_PATTERN = Pattern.compile("v(\\d+)/");
        
        private int apiVersion;
        
        public ApiVersionCondition(int apiVersion){
            this.apiVersion = apiVersion;
        }
        
        public ApiVersionCondition combine(ApiVersionCondition other) {
            // 采用最后定义优先原则，则方法上的定义覆盖类上面的定义
            return new ApiVersionCondition(other.getApiVersion());
        }
    
        public ApiVersionCondition getMatchingCondition(HttpServletRequest request) {
    //    	String pathInfo = request.getPathInfo();//这个方法获取是null，报错。
        	String path = request.getServletPath(); 
        	if (path == null) {
    			return null;
    		}
            Matcher m = VERSION_PREFIX_PATTERN.matcher(path);//匹配路径
            if(m.find()){
                Integer version = Integer.valueOf(m.group(1));
                if(version >= this.apiVersion) // 如果请求的版本号大于配置版本号， 则满足
                    return this;
            }
            return null;
        }
    
        public int compareTo(ApiVersionCondition other, HttpServletRequest request) {
            // 优先匹配最新的版本号
            return other.getApiVersion() - this.apiVersion;
        }
    
        public int getApiVersion() {
            return apiVersion;
        }
    
    }

3.演示：

{%asset_img d.png%}

--------------------------------------

{%asset_img e.png%}

   