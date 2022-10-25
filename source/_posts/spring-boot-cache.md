---
title: Spring-Cache使用
date: 2018-03-23 17:29:57
categories: spring-boot
tags: spring cache j2cache JetCache redis-cache
---

缓存在系统中的作用举足轻重，本文记录各种缓存方案在web系统开发中的应用，以备不时查阅。

## 缓存的介绍

所谓缓存，就是把一些必要的数据从硬件存储或者网络等写入到缓存寄存器或者内存中，以便加快访问速度。加了缓存，应用的使用体验会有一个量级的提升。

参考：https://m.w3cschool.cn/article/29307174.html

缓存大概原理图：

{% asset_img a-2.png %}

## spring cache的一些基本介绍

### 基本使用

先在pom中引入依赖：
```xml
<!--加入spring cache-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

- 第1步：在启动类上加注解@EnableCaching
- 第2步：在具体方法上加注解【@CachePut、@CacheEvict、@Caching】

### 常用注解

- @EnableCaching
- @Cacheable
- @CachePut
- @CacheEvict
- @Caching
- @CacheConfig

```text
@Cacheable：主要用于 “查询” 功能
@CachePut：主要用于 “修改” 功能
@CacheEvict：主要用于 “删除” 功能
```

#### @Cacheable

@Cacheable注解有下面9个属性value、 cacheNames、 key、 keyGenerator、 cacheManager、 cacheResolver、 condition、 unless、 sync。接下来我们就分别来介绍一下它的使用。

1. value和cacheNames属性

两者是一样的，标志缓存数据要存入那个缓存名称下的缓存器。缓存名称。定义为数组，可以是多个名称。

这里缓存名称，就是分片或者分组的盖面，每个名称下就是一个缓存器。

```text
@Cacheable(cacheNames = "userInfo") //@Cacheable(value = "userInfo")
UserInfo get(Integer userId);
```
上面配置是等级的。

2. key属性

该属性就是指定缓存的key值，默认是用方法调用传过来的参数作为key。缓存中存储内容的格式为：Entry<key,value> 形式。

- 如果请求没有参数：key=new SimpleKey()；
- 如果请求有一个参数：key=参数的值
- 如果请求有多个参数：key=newSimpleKey(params)； (你只要知道 key不会为空就行了)

key的设定，也可以自定义，可以使用spring的SpEL表达式来编写，也可以使用自定义的key生成器 keyGenerator。

下面是一些SpEL表达式定义key：

{% asset_img a-1.png %}

3. keyGenerator 属性

key/keyGenerator属性：二选一使用。   

下面是自定义key生成器：

```text
/**
 * 缓存自定义key生成器
 * @return
 */
@Bean("myKeyGenerator")
public KeyGenerator keyGenerator(){
    return new KeyGenerator(){

        @Override
        public Object generate(Object target, Method method, Object... params) {
            return method.getName()+ Arrays.asList(params).toString();
        }
    };
}

/**
 * 支持 lambda 表达式编写
 */
/*@Bean("myKeyGenerator")
public KeyGenerator keyGenerator(){
    return ( target,  method, params)-> method.getName()+ Arrays.asList(params).toString();
}*/
```

引用：

```text
//@Cacheable(cacheNames = "userInfo", key = "#root.methodName.#userId") //@Cacheable(value = "userInfo")
@Cacheable(cacheNames = "userInfo", keyGenerator = "myKeyGenerator") //myKeyGenerator为该bean的名称
@Override
public UserInfo get(Integer userId) {
    cacheManager.getCacheNames().forEach(s -> {
        System.out.println(">>>>cacheName:" + s);
    });
    System.out.println("获取用户信息:" + userId);
    return (UserInfo) userInfoMap.get(userId);
}
```

4. cacheManager属性

该属性，指定缓存管理器。针对不同的缓存技术，需要实现不同的cacheManager，spring为我们定义了下面一些缓存管理器：

|       CacheManager        |            描述             |
|:-------------------------:|:-------------------------:|
|    SimpleCacheManager     | 使用简单的Collection来缓存，主要用于测试 |
| ConcurrentMapCacheManager | 使用ConcurrentMap作为缓存技术，默认  |
|    EhCacheCacheManager    |    使用EhCache缓存框架技术作为缓存    |
|     GuavaCacheManager     | 使用google guava中的缓存技术实现缓存  |
|   HazelcastCacheManager   |      使用Hazelcast技术缓存      |
|    JCacheCacheManager     |     使用JCache标准实现的缓存技术     |
|     RedisCacheManager     |       使用Redis作为缓存技术       |

5. cacheResolver属性

该属性，用来指定缓存解析器。使用配置同 cacheManager 类似，可自行百度。（cacheManager指定管理器/cacheResolver指定解析器 它俩也是二选一使用）

6. condition 属性

该属性表示当达成它设定的条件的时候才写入缓存。示例如下：
```java
@Cacheable(cacheNames = "userInfo", key = "#root.methodName.#userId", condition = "#userId == 2") //设置condition缓存条件，当用户id == 2的时候才缓存
@Override
public UserInfo get(Integer userId) {
    cacheManager.getCacheNames().forEach(s -> {
        System.out.println(">>>>cacheName:" + s);
    });
    System.out.println("获取用户信息:" + userId);
    return (UserInfo) userInfoMap.get(userId);
}
```
```java
@Cacheable(value = "user",condition = "#a0>1 and #root.methodName eq 'getUser'")//传入的第一个参数的值>1 且 方法名为 getUser 的时候才进行缓存
User getUser(Integer id);
```

7. unless属性

从该属性名字就可以看出来，当不满足条件时候才写入缓存。

```java
@Cacheable(cacheNames = "userInfo", key = "#root.methodName.#userId", unless = "#userId == 2") //属性unless表示，当用户id不等于2的写入缓存
@Override
public UserInfo get(Integer userId) {
    cacheManager.getCacheNames().forEach(s -> {
        System.out.println(">>>>cacheName:" + s);
    });
    System.out.println("获取用户信息:" + userId);
    return (UserInfo) userInfoMap.get(userId);
}
```

```java
@Cacheable(value = "user",unless = "#result == null")//当方法返回值为 null 时，就不缓存
User getUser(Integer id);

@Cacheable(value = "user",unless = "#a0 == 1")//如果第一个参数的值是1,结果不缓存
User getUser(Integer id);
```

8. sync属性

该属性用来指定是否使用异步模式，该属性默认值为`false`，默认为同步模式。异步模式指定`sync = true`即可，异步模式下`unless`属性不可用。

sync：当值为true时，相当于添加了本地锁，可以有效地解决缓存击穿问题

####  @CachePut

该注解表示，不管缓存有没有，都会把方法返回的结果写入到缓存中。如果缓存已经存在，那么该key的缓存将会被覆盖，所以适用于缓存更新。

官方强烈建议不要把注解`@Cacheable`和注解`@CachePut`放在同一个方法中。

```java
//添加缓存注解CachePut，表示更新底层数据库数据后，接着更新该key的缓存
@CachePut(cacheNames = "userInfo", key = "#root.methodName.#userId")
@Override
public UserInfo update(Integer userId, String userName) {
    System.out.println("开始更新用户");
    UserInfo userInfo = (UserInfo) userInfoMap.get(userId);
    userInfo.setName(userName);
    userInfoMap.put(userId, userInfo);
    return userInfo;
}
```

该注解和注解`@Cacheable`中的属性差不多，可以参考其使用方法。

####  @CacheEvict

这个就是我们理解的删除缓存。

`@CacheEvict`是用来标注在需要清除缓存元素的方法或类上的。当标记在一个类上时表示其中所有的方法的执行都会触发缓存的清除操作。

代码示例：
```java
//缓存失效
@CacheEvict(cacheNames = "userInfo", key = "#root.methodName.#userId")
@Override
public UserInfo save(Integer userId, String userName, Integer age) {
    System.out.println("开始添加用户");
    userInfoMap.put(userId, new UserInfo(userId, userName, age));
    return (UserInfo) userInfoMap.get(userId);
}
```

该缓存注解相比`@Cacheable`和`@CachePut`,增加了两个属性：
```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface CacheEvict {
    @AliasFor("cacheNames")
    String[] value() default {};

    @AliasFor("value")
    String[] cacheNames() default {};

    String key() default "";

    String keyGenerator() default "";

    String cacheManager() default "";

    String cacheResolver() default "";

    String condition() default "";

    boolean allEntries() default false;

    boolean beforeInvocation() default false;
}
```

1. allEntries属性

是否需要清除缓存中的所有元素。默认为`false`，表示不需要。当指定了`allEntries`为`true`时，`Spring Cache`将忽略指定的`key`，删除缓存中所有键；

2. beforeInvocation属性

是否在方法执行成功之后触发键删除操作，默认是在对应方法成功执行之后触发的，若此时方法抛出异常而未能成功返回，不会触发清除操作。指定该属性值为`true`时，Spring会在调用该方法之前清除缓存中的指定元素。

#### @Caching（不常用）

@Caching 注解可以在一个方法或者类上同时指定多个Spring Cache相关的注解。

其拥有三个属性：cacheable、put 和 evict，分别用于指定@Cacheable、@CachePut 和 @CacheEvict。对于一个数据变动，更新多个缓存的场景，可以通过 @Caching 来实现：
```java
@Caching(cacheable = @Cacheable(cacheNames = "caching", key = "#age"), evict = @CacheEvict(cacheNames = "t4", key = "#age"))
public String caching(int age) {
    return "caching: " + age + "-->" + UUID.randomUUID().toString();
}
```
上面组合操作：
- 从`caching::age`缓存取数据，不存在时执行方法并写入缓存
- 失效缓存`t4::age`

#### @CacheConfig（不常用）

如果一个类中，多个方法都有同样的`cacheName`，`keyGenerator`，`cacheManager`和`cacheResolver`，可以直接使用`@CacheConfig`注解在类上声明，这个类中的方法都会使用`@CacheConfig`属性设置的相关配置。
```java
@Component
@CacheConfig(cacheNames = "mall_cache")
public class CacheComponent {

	
    @Cacheable(key = "'perm-whitelist-'+#clientId", unless="#result == null")
    public List<String> cacheWriteList(String clientId){
    	...
    }
       
     @Cacheable(key = "'perm-cutom-aci-' + #tenantId + '-' + #roleId + '-' + #tenantLevel + '-' + #subType", unless="#result == null")
    public List<RequestDto> cacheRequest(Long tenantId,Long roleId,Integer tenantLevel,Integer subType){
        ...
    }
}

```

### 原理

原理，原理，原理……


## spring-cache集成Ehcache做本地缓存

[Ehcache官网](http://ehcache.org/)

## spring-cache集成Caffeine本地缓存框架

[Caffeine](https://github.com/ben-manes/caffeine)

1. 引入相关jar：
```text
<!--加入spring cache-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
<!--集成本地缓存框架caffeine-->
<dependency>
    <groupId>com.github.ben-manes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
    <version>3.1.1</version>
</dependency>
```

2. 配置类：
```java
package com.example.demo;

import com.github.benmanes.caffeine.cache.Caffeine;
import org.springframework.cache.CacheManager;
import org.springframework.cache.caffeine.CaffeineCacheManager;
import org.springframework.cache.interceptor.KeyGenerator;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.lang.reflect.Method;
import java.util.Arrays;
import java.util.concurrent.TimeUnit;

@Configuration
public class CacheConfig {

    /**
     * 缓存自定义key生成器
     * @return
     */
    @Bean("myKeyGenerator")
    public KeyGenerator keyGenerator(){
        return new KeyGenerator(){

            @Override
            public Object generate(Object target, Method method, Object... params) {
                return method.getName()+ Arrays.asList(params).toString();
            }
        };
    }

    /**
     * 支持 lambda 表达式编写
     */
    /*@Bean("myKeyGenerator")
    public KeyGenerator keyGenerator(){
        return ( target,  method, params)-> method.getName()+ Arrays.asList(params).toString();
    }*/

    /**
     * 配置缓存管理器
     * @return 缓存管理器
     */
    @Bean("caffeineCacheManager")
    public CacheManager cacheManager() {
        CaffeineCacheManager cacheManager = new CaffeineCacheManager();
        cacheManager.setCaffeine(Caffeine.newBuilder()
                // 最后一次写入后经过固定时间过期300秒
                .expireAfterWrite(100, TimeUnit.SECONDS)
                // 初始的缓存空间大小
                .initialCapacity(100)
                // 缓存的最大条数
                .maximumSize(1000));
        return cacheManager;
    }

    /**
     * 相当于在构建LoadingCache对象的时候 build()方法中指定过期之后的加载策略方法
     * 必须要指定这个Bean，refreshAfterWrite=60s属性才生效
     * @return
     */
    /*@Bean
    public CacheLoader<String, Object> cacheLoader() {
        CacheLoader<String, Object> cacheLoader = new CacheLoader<String, Object>() {
            @Override
            public Object load(String key) throws Exception {
                return null;
            }
            // 重写这个方法将oldValue值返回回去，进而刷新缓存
            @Override
            public Object reload(String key, Object oldValue) throws Exception {
                return oldValue;
            }
        };
        return cacheLoader;
    }

    @Bean
    public Cache<String, Object> cache(CacheLoader<String, Object> cacheLoader) {
        return Caffeine.newBuilder()
                // 创建缓存或者最近一次更新缓存后经过固定的时间间隔，刷新缓存 最后一次写入后经过固定时间过期
                .refreshAfterWrite(5, TimeUnit.SECONDS)
                // 初始的缓存空间大小
                .initialCapacity(100)
                // 缓存的最大条数
                .maximumSize(10000)
                //软引用
                .softValues()
                .build(cacheLoader);
    }*/
}

```

## spring-cache-redis分布式缓存

#### 添加相关依赖

在`pom.xml`文件添加：
```xml
<!--spring cache 集成redis做缓存-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <exclusions>
        <!--排除lettuce客户端（默认使用lettuce客户端）-->
        <exclusion>
            <groupId>io.lettuce</groupId>
            <artifactId>lettuce-core</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<!--redis连接池-->
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
</dependency>
```

#### redis配置

在`application.yml`添加`redis`配置文件：
```yaml
spring:
  redis:
    database: 0
    host: 192.168.0.106
    port: 6380
    password: a1234567
    jedis:
      pool:
        # 连接池中的最大空闲连接 默认8
        max-idle: 8
        # 连接池中的最小空闲连接 默认0
        min-idle: 0
        # 连接池最大连接数 默认8 ，负数表示没有限制
        max-active: 8
        # 连接池最大阻塞等待时间（使用负值表示没有限制） 默认-1
        max-wait: -1
    timeout: 3000
  cache:
    type: redis   # 指定使用的缓存类型
      # redis:    当自定义ChacheManager时，就这里的配置不需要配置，配置了也不起作用
      #   use-key-prefix: true
      #  key-prefix: "demo:"
      #  time-to-live: 60000  #缓存超时时间 单位：ms
      #  cache-null-values: false #是否缓存空值
    cache-names: user
cache:
  ttl: '{"user":60,"dept":30}'  #自定义某些缓存空间的过期时间

```

#### 缓存配置类

添加配置类如下：
```java
package com.example.demo;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.cache.CacheManager;
import org.springframework.cache.interceptor.KeyGenerator;
import org.springframework.cache.interceptor.SimpleKeyGenerator;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheConfiguration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.RedisSerializationContext;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import java.time.Duration;
import java.util.HashMap;
import java.util.Map;
import java.util.Set;

@Configuration
public class RedisConfiguration {

    // ${cache} 获取配置文件的配置信息   #{}是spring表达式，获取Bean对象的属性
    @Value("#{${cache.ttl}}")
    private Map<String, Long> ttlParams;

    /**
     * @param redisConnectionFactory
     * @功能描述 redis作为缓存时配置缓存管理器CacheManager，主要配置序列化方式、自定义
     * <p>
     * 注意：配置缓存管理器CacheManager有两种方式：
     * 方式1：通过RedisCacheConfiguration.defaultCacheConfig()获取到默认的RedisCacheConfiguration对象，
     * 修改RedisCacheConfiguration对象的序列化方式等参数【这里就采用的这种方式】
     * 方式2：通过继承CachingConfigurerSupport类自定义缓存管理器，覆写各方法，参考：
     * https://blog.csdn.net/echizao1839/article/details/102660649
     * <p>
     * 切记：在缓存配置类中配置以后，yaml配置文件中关于缓存的redis配置就不会生效，如果需要相关配置需要通过@value去读取
     */
    @Bean
    public CacheManager cacheManager(RedisConnectionFactory redisConnectionFactory) {
        RedisCacheConfiguration redisCacheConfiguration = RedisCacheConfiguration.defaultCacheConfig();
        redisCacheConfiguration = redisCacheConfiguration
                // 设置key采用String的序列化方式
                .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(StringRedisSerializer.UTF_8))
                //设置value序列化方式采用jackson方式序列化
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(serializer()))
                //当value为null时不进行缓存
                .disableCachingNullValues()
                // 配置缓存空间名称的前缀
                .prefixCacheNameWith("demo:")
                //全局配置缓存过期时间【可以不配置】
                .entryTtl(Duration.ofMinutes(30L));
        //专门指定某些缓存空间的配置，如果过期时间【主要这里的key为缓存空间名称】
        Map<String, RedisCacheConfiguration> map = new HashMap<>();
        Set<Map.Entry<String, Long>> entries = ttlParams.entrySet();
        for (Map.Entry<String, Long> entry : entries) {
            //指定特定缓存空间对应的过期时间
//            map.put("user", redisCacheConfiguration.entryTtl(Duration.ofSeconds(10)));
            map.put(entry.getKey(), redisCacheConfiguration.entryTtl(Duration.ofSeconds(entry.getValue())));
        }
        return RedisCacheManager
                .builder(redisConnectionFactory)
                .cacheDefaults(redisCacheConfiguration)  //默认配置
                .withInitialCacheConfigurations(map)  //某些缓存空间的特定配置
                .build();
    }


    /**
     * 自定义缓存的redis的KeyGenerator【key生成策略】
     * 注意: 该方法只是声明了key的生成策略,需在@Cacheable注解中通过keyGenerator属性指定具体的key生成策略
     * 可以根据业务情况，配置多个生成策略
     * 如: @Cacheable(value = "key", keyGenerator = "cacheKeyGenerator")
     */
    @Bean
    public KeyGenerator keyGenerator() {
        /**
         * target: 类
         * method: 方法
         * params: 方法参数
         */
        return (target, method, params) -> {
            //获取代理对象的最终目标对象
            StringBuilder sb = new StringBuilder();
            sb.append(target.getClass().getSimpleName()).append(":");
            sb.append(method.getName()).append(":");
            //调用SimpleKey的key生成器
            Object key = SimpleKeyGenerator.generateKey(params);
            return sb.append(key);
        };
    }


    /**
     * @param redisConnectionFactory：配置不同的客户端，这里注入的redis连接工厂不同： JedisConnectionFactory、LettuceConnectionFactory
     * @功能描述 ：配置Redis序列化，原因如下：
     * （1） StringRedisTemplate的序列化方式为字符串序列化，
     * RedisTemplate的序列化方式默为jdk序列化（实现Serializable接口）
     * （2） RedisTemplate的jdk序列化方式在Redis的客户端中为乱码，不方便查看，
     * 因此一般修改RedisTemplate的序列化为方式为JSON方式【建议使用GenericJackson2JsonRedisSerializer】
     */
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        GenericJackson2JsonRedisSerializer genericJackson2JsonRedisSerializer = serializer();
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        // key采用String的序列化方式
        redisTemplate.setKeySerializer(StringRedisSerializer.UTF_8);
        // value序列化方式采用jackson
        redisTemplate.setValueSerializer(genericJackson2JsonRedisSerializer);
        // hash的key也采用String的序列化方式
        redisTemplate.setHashKeySerializer(StringRedisSerializer.UTF_8);
        //hash的value序列化方式采用jackson
        redisTemplate.setHashValueSerializer(genericJackson2JsonRedisSerializer);
        redisTemplate.setConnectionFactory(redisConnectionFactory);
        return redisTemplate;
    }

    /**
     * 此方法不能用@Ben注解，避免替换Spring容器中的同类型对象
     */
    public GenericJackson2JsonRedisSerializer serializer() {
        return new GenericJackson2JsonRedisSerializer();
    }
}


```

#### 开启缓存

````java
@EnableCaching //开启缓存
@SpringBootApplication
public class SpringRedisCacheApp {

    public static void main(String[] args) {
        SpringApplication.run(SpringRedisCacheApp.class, args);
    }
}
````

#### 使用

使用遵循`spring cache`规范，参考上面使用方法，这里不再重复介绍。


## 缓存框架JetCache

与Spring Cache类似，JetCache提供了一套操作缓存的API，可以同时支持本地和分布式缓存，但是不能支持缓存同步更新。

[JetCache](https://github.com/alibaba/jetcache)

## 二级缓存框架J2Cache

J2Cache是开源中国研发使用的一个独立的二级缓存框架，支持本地环境和分布式环境，并且支持缓存同步。
解决频繁访问集中式缓存带来的带宽压力，相同服务的多节点缓存同步问题。

[J2Cache](https://gitee.com/ld/J2Cache)

## 问题

这里收集一些使用缓存中出现的一些常见问题。

### 缓存时间如何设置

缓存的时间该设置多长呢，多长才是最合理的，考虑的因素是什么？？？

### 如何对每个缓存设置不同的缓存时间

实际使用中，并不是所有的缓存都统一设置成一个相同的失效时间，根据业务的需要，可能要对不同的缓存设置不同的失效时间，那么该怎么设置呢？？？？

### 读模式：缓存穿透，缓存击穿，缓存雪崩

缓存穿透：查询一个null数据。解决：缓存空数据：cache-null-values=true

缓存击穿：大量并发请求进来同时查询一个正好过期的数据。 解决： 加锁

缓存雪崩：大量的key同时过期 解决：加随机时间

### 写模式 （如何保证缓存和数据库一致性）

1）加锁模式

2）引入canal
