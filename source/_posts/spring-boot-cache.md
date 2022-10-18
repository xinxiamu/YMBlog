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

### 原理

dddd


## spring-cache集成Ehcache做本地缓存

[Ehcache官网](http://ehcache.org/)

## spring-cache集成Caffeine本地缓存框架

[Caffeine](https://github.com/ben-manes/caffeine)

## spring-cache-redis分布式缓存


## 缓存框架JetCache

与Spring Cache类似，JetCache提供了一套操作缓存的API，可以同时支持本地和分布式缓存，但是不能支持缓存同步更新。

[JetCache](https://github.com/alibaba/jetcache)

## 二级缓存框架J2Cache

J2Cache是开源中国研发使用的一个独立的二级缓存框架，支持本地环境和分布式环境，并且支持缓存同步。
解决频繁访问集中式缓存带来的带宽压力，相同服务的多节点缓存同步问题。

[J2Cache](https://gitee.com/ld/J2Cache)
