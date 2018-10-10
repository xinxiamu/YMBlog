---
title: shiro安全框架入门
date: 2018-05-14 11:22:37
categories: shiro
tags: shiro入门
---

Apache Shiro([官网](http://shiro.apache.org))是个java的安全框架，轻便、api简单、功能全面的特性获得 许多开发者的青睐。其提供了认证、授权、加密以及会话管理等功能……

## 介绍

#### 1.功能特性
Shiro 包含 10 个内容，如下图：

{%asset_img shiro-01.png%}

- Authentication: 身份认证，拥有合法身份才能登录系统并使用。
- Authorization： 授权验证。即认证某个已经得到身份认证的用户是否拥有对某个功能或者某种资源的使用权限。
- Session Manager： 会话管理。用户登录后，没退出之前，为一次会话，所有的信息都在会话中。会话可以是javase环境，也可以是java web环境。
- Cryptography： 加密，保护数据的安全性。
- Web Support： web支持，把shiro容易的集成到web环境中。
- Caching： 缓存，用户登陆后，用户的信息、拥有的角色权限不必每次查，缓存起来。
- Concurrency： shiro 支持多线程应用的并发验证，即如在一个线程中开启另一个线程，能把权限自动传播过去。
- Testing：提供测试支持。
- Run As：允许一个用户假装为另一个用户（如果他们允许）的身份进行访问。
- Remember Me：记住我，这个是非常常见的功能，即一次登录后，下次再来的话不用登录了。

#### 2.运行原理
1.原理图1（应用程序角度）：
{%asset_img shiro-02.png%}

- Subject：主体，代表当前“用户”，这里的用户指的不仅仅是登录用户，指的是任何与该系统交互的主体。所有的Subject都和SecurityManager绑定，与Subject的任何交互都委托给SecurityManager来处理。
- SecurityManager： 安全管理器，是shiro的核心功能。它负责所有的安全操作管理，管理这所有的Subject。类似于SpringMvc中的DispatcherServlet控制器。
- Realm： 域。所有的安全数据保存，获取都要通过Realm。SecurityManager和它交互，获取相关数据。Realm就类似于数据源DataSource。

2.原理图2（内部架构）：
{%asset_img shiro-03.png%}

- Subject：主体。
- SecurityManager：安全管理器。
- Authenticator： 认证器，负责对主体的身份认证。可以自定义，重新设定认证策略，即什么情况下才算认证通过。
- Authrizer： 授权器，或者访问控制器。决定主体是否有权限进行相应操作。
- Realm： 可以一个或者多个。可以是jdbc，redis，内存等实现。
- SessionManager： 会话管理器。
- SessionDAO： 数据访问对象，用于会话的CRUD。可以自定义，控制session存储的位置，关系数据库，redis等。另外，可以使用缓存，提高性能。
- CacheManager： 缓存管理器。
- Cryptography： 密码模块。提供了一些加解密算法。

#### 3.过滤器
应用到web系统中时，Shiro会默认创建一些过滤器对客户端请求进行过滤。常用过滤器有：

|  过滤器简称   | 对应的 Java 类 |
| :------| :------|
| anon | org.apache.shiro.web.filter.authc.AnonymousFilter |
| authc | org.apache.shiro.web.filter.authc.FormAuthenticationFilter |
| authcBasic | org.apache.shiro.web.filter.authc.BasicHttpAuthenticationFilter |
| perms | org.apache.shiro.web.filter.authz.PermissionsAuthorizationFilter |
| port | org.apache.shiro.web.filter.authz.PortFilter |
| rest | org.apache.shiro.web.filter.authz.HttpMethodPermissionFilter |
| roles | org.apache.shiro.web.filter.authz.RolesAuthorizationFilter |
| ssl | org.apache.shiro.web.filter.authz.SslFilter |
| user | org.apache.shiro.web.filter.authc.UserFilter |
|  logout | org.apache.shiro.web.filter.authc.LogoutFilter |
| noSessionCreation | org.apache.shiro.web.filter.session.NoSessionCreationFilter |

说明：

    /admins/**=anon               # 表示该 uri 可以匿名访问
    /admins/**=auth               # 表示该 uri 需要认证才能访问
    /admins/**=authcBasic         # 表示该 uri 需要 httpBasic 认证
    /admins/**=perms[user:add:*]  # 表示该 uri 需要认证用户拥有 user:add:* 权限才能访问
    /admins/**=port[8081]         # 表示该 uri 需要使用 8081 端口
    /admins/**=rest[user]         # 相当于 /admins/**=perms[user:method]，其中，method 表示  get、post、delete 等
    /admins/**=roles[admin]       # 表示该 uri 需要认证用户拥有 admin 角色才能访问
    /admins/**=ssl                # 表示该 uri 需要使用 https 协议
    /admins/**=user               # 表示该 uri 需要认证或通过记住我认证才能访问
    /logout=logout                # 表示注销,可以当作固定配置
    
 _注意_
 anon，authcBasic，auchc，user 是认证过滤器。
 perms，roles，ssl，rest，port 是授权过滤器。   

## 认证

1.认证路程图：
{%asset_img shiro-04.png%} 

2.使用代码示例：

```java
package com.example.springboot2shirostart;

import org.apache.shiro.SecurityUtils;
import org.apache.shiro.authc.UsernamePasswordToken;
import org.apache.shiro.mgt.DefaultSecurityManager;
import org.apache.shiro.realm.SimpleAccountRealm;
import org.apache.shiro.subject.Subject;
import org.junit.Before;
import org.junit.Test;

/**
 * 简单认证
 */
public class AuthenticationTest {

    SimpleAccountRealm simpleAccoutRealm = new SimpleAccountRealm();

    @Before
    public void addUser() {
        simpleAccoutRealm.addAccount("zmt","123456");
//        simpleAccoutRealm.addAccount("zmt1","123456");
    }

    @Test
    public void  testAuthentication() {
        //1.创建securityManager环境
        DefaultSecurityManager defaultSecurityManager = new DefaultSecurityManager();
        defaultSecurityManager.setRealm(simpleAccoutRealm);

        //2.主体提交认证请求
        SecurityUtils.setSecurityManager(defaultSecurityManager);
        Subject subject = SecurityUtils.getSubject();

        UsernamePasswordToken token = new UsernamePasswordToken("zmt","123456");
        subject.login(token);
        System.out.println("isAuthenticated:" + subject.isAuthenticated());//已认证

        subject.logout();//退出登录
        System.out.println("isAuthenticated:" + subject.isAuthenticated());//未认证

    }
}

```

## 授权

1.流程图：
{%asset_img shiro-05.png%}
