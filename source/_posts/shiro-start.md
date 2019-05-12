---
title: shiro安全框架入门
date: 2018-05-14 11:22:37
categories: shiro
tags: shiro入门
---

Apache Shiro([官网](http://shiro.apache.org))是个java的安全框架，轻便、api简单、功能全面的特性获得 许多开发者的青睐。其提供了认证、授权、加密以及会话管理等功能……

### 一.介绍

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

### 二.认证

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

### 三.授权

1.流程图：
{%asset_img shiro-05.png%}

2。代码示例：

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
//        simpleAccoutRealm.addAccount("zmt","123456");
//        simpleAccoutRealm.addAccount("zmt1","123456");

        simpleAccoutRealm.addAccount("zmt","123456","admin","user"); //添加角色
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

        subject.checkRole("admin"); //admin通过，admin1报错
        subject.checkRoles("admin","user1"); //拥有该两个角色

//        subject.logout();//退出登录
//        System.out.println("isAuthenticated:" + subject.isAuthenticated());//未认证

    }


}

```



### 四.Shiro自定义Realm

#### 1.IniRealm讲解

代码样例：

```java
package com.example.springboot2shirostart;

import org.apache.shiro.SecurityUtils;
import org.apache.shiro.authc.UsernamePasswordToken;
import org.apache.shiro.mgt.DefaultSecurityManager;
import org.apache.shiro.realm.text.IniRealm;
import org.apache.shiro.subject.Subject;
import org.junit.Test;

/**
 * 简单认证
 */
public class IniRealmTest {

    @Test
    public void  test1() {

        IniRealm iniRealm = new IniRealm("classpath:user.ini");

        //1.创建securityManager环境
        DefaultSecurityManager defaultSecurityManager = new DefaultSecurityManager();
        defaultSecurityManager.setRealm(iniRealm);

        //2.主体提交认证请求
        SecurityUtils.setSecurityManager(defaultSecurityManager);
        Subject subject = SecurityUtils.getSubject();

        UsernamePasswordToken token = new UsernamePasswordToken("zmt","123456");
        subject.login(token);
        System.out.println("isAuthenticated:" + subject.isAuthenticated());//已认证

        subject.checkRole("admin"); //admin通过，admin1报错
        subject.checkRoles("admin","user"); //拥有该两个角色

        //权限
        subject.checkPermission("user:delete");
//        subject.checkPermission("user:insert"); //报错，么有insert权限
        subject.checkPermission("user:update"); //报错，没有update权限
    }


}

```

```text
编辑：user.ini

[users]
zmt=123456,admin,user
[roles]
admin=user:delete,user:update
```
#### 2.JdbcRealm讲解

用户、角色、权限数据都在数据库里面。而不是在配置文件里面。 
两种方式，一种是使用默认查询语句；另外一种是自己设计表，写sql语句，不必遵守默认的。   

##### 2.1.默认方式

- 首先，创建相关默认数据表：   
```sql
/*
Navicat MySQL Data Transfer

Source Server         : localhost
Source Server Version : 50718
Source Host           : localhost:3307
Source Database       : shiro-test

Target Server Type    : MYSQL
Target Server Version : 50718
File Encoding         : 65001

Date: 2018-10-15 17:09:45
*/

SET FOREIGN_KEY_CHECKS=0;

-- ----------------------------
-- Table structure for roles_permissions
-- ----------------------------
DROP TABLE IF EXISTS `roles_permissions`;
CREATE TABLE `roles_permissions` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `role_name` varchar(255) NOT NULL,
  `permission` varchar(255) NOT NULL,
  PRIMARY KEY (`id`),
  KEY `role_name` (`role_name`),
  CONSTRAINT `roles_permissions_ibfk_1` FOREIGN KEY (`role_name`) REFERENCES `user_roles` (`role_name`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4;

-- ----------------------------
-- Records of roles_permissions
-- ----------------------------
INSERT INTO `roles_permissions` VALUES ('1', 'admin', 'user:select');

-- ----------------------------
-- Table structure for user_roles
-- ----------------------------
DROP TABLE IF EXISTS `user_roles`;
CREATE TABLE `user_roles` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(255) NOT NULL,
  `role_name` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `username` (`username`),
  KEY `role_name` (`role_name`),
  CONSTRAINT `user_roles_ibfk_1` FOREIGN KEY (`username`) REFERENCES `users` (`username`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4;

-- ----------------------------
-- Records of user_roles
-- ----------------------------
INSERT INTO `user_roles` VALUES ('1', 'zmt', 'admin');

-- ----------------------------
-- Table structure for users
-- ----------------------------
DROP TABLE IF EXISTS `users`;
CREATE TABLE `users` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(255) NOT NULL,
  `password` varchar(255) NOT NULL,
  `password_salt` varchar(255) NOT NULL,
  PRIMARY KEY (`id`),
  KEY `username` (`username`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4;

-- ----------------------------
-- Records of users
-- ----------------------------
INSERT INTO `users` VALUES ('1', 'zmt', '123456', 'abcd');
SET FOREIGN_KEY_CHECKS=1;

```
- 代码样例：
```java
package com.example.springboot2shirostart;

import com.alibaba.druid.pool.DruidDataSource;
import org.apache.shiro.SecurityUtils;
import org.apache.shiro.authc.UsernamePasswordToken;
import org.apache.shiro.mgt.DefaultSecurityManager;
import org.apache.shiro.realm.jdbc.JdbcRealm;
import org.apache.shiro.subject.Subject;
import org.junit.Test;

/**
 * 简单认证
 */
public class JdbcRealmTest {

    DruidDataSource dataSource = new DruidDataSource();

    {
        dataSource.setUrl("jdbc:mysql://localhost:3307/shiro-test");
        dataSource.setUsername("root");
        dataSource.setPassword("root");
    }

    /**
     * 查询默认的数据表做相关操作。
     */
    @Test
    public void  test1() {

        JdbcRealm jdbcRealm = new JdbcRealm();
        jdbcRealm.setDataSource(dataSource); //设置数据源
        jdbcRealm.setPermissionsLookupEnabled(true); //是否可以查看权限。默认为false。不打开，则无法查看权限

        //1.创建securityManager环境
        DefaultSecurityManager defaultSecurityManager = new DefaultSecurityManager();
        defaultSecurityManager.setRealm(jdbcRealm);

        //2.主体提交认证请求
        SecurityUtils.setSecurityManager(defaultSecurityManager);
        Subject subject = SecurityUtils.getSubject();

        UsernamePasswordToken token = new UsernamePasswordToken("zmt","123456");//用户
        subject.login(token);
        System.out.println("isAuthenticated:" + subject.isAuthenticated());//已认证

        subject.checkRole("admin"); //admin通过，admin1报错
//        subject.checkRoles("admin","user"); //拥有该两个角色

        //权限
        subject.checkPermission("user:select");//jdbcRealm.setPermissionsLookupEnabled(true);
//        subject.checkPermission("user:insert"); //报错，么有insert权限
//        subject.checkPermission("user:update"); //报错，没有update权限
    }


}

```

##### 2.2.自己设计表写查询语句

- 创建数据表 
不必遵循默认规范，表名、字段名都可以自定义。  

脚本： 
```sql
/*
Navicat MySQL Data Transfer

Source Server         : localhost
Source Server Version : 50718
Source Host           : localhost:3307
Source Database       : shiro-test

Target Server Type    : MYSQL
Target Server Version : 50718
File Encoding         : 65001

Date: 2018-10-17 11:42:25
*/

SET FOREIGN_KEY_CHECKS=0;

-- ----------------------------
-- Table structure for test_role_permissions
-- ----------------------------
DROP TABLE IF EXISTS `test_role_permissions`;
CREATE TABLE `test_role_permissions` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `role_name` varchar(255) NOT NULL,
  `permission` varchar(255) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4;

-- ----------------------------
-- Records of test_role_permissions
-- ----------------------------
INSERT INTO `test_role_permissions` VALUES ('1', 'caiwu', 'user:update');

-- ----------------------------
-- Table structure for test_user
-- ----------------------------
DROP TABLE IF EXISTS `test_user`;
CREATE TABLE `test_user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `user_name` varchar(255) NOT NULL,
  `password` varchar(255) NOT NULL,
  PRIMARY KEY (`id`),
  KEY `username` (`user_name`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4;

-- ----------------------------
-- Records of test_user
-- ----------------------------
INSERT INTO `test_user` VALUES ('1', 'xr', '123456');

-- ----------------------------
-- Table structure for test_user_role
-- ----------------------------
DROP TABLE IF EXISTS `test_user_role`;
CREATE TABLE `test_user_role` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `test_user_name` varchar(255) NOT NULL,
  `role_name` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4;

-- ----------------------------
-- Records of test_user_role
-- ----------------------------
INSERT INTO `test_user_role` VALUES ('1', 'xr', 'caiwu');
SET FOREIGN_KEY_CHECKS=1;
```
- 代码样例：
```java
package com.example.springboot2shirostart;

import com.alibaba.druid.pool.DruidDataSource;
import org.apache.shiro.SecurityUtils;
import org.apache.shiro.authc.UsernamePasswordToken;
import org.apache.shiro.mgt.DefaultSecurityManager;
import org.apache.shiro.realm.jdbc.JdbcRealm;
import org.apache.shiro.subject.Subject;
import org.junit.Test;

/**
 * 简单认证
 */
public class JdbcRealmTest {

    DruidDataSource dataSource = new DruidDataSource();

    {
        dataSource.setUrl("jdbc:mysql://localhost:3307/shiro-test");
        dataSource.setUsername("root");
        dataSource.setPassword("root");
    }

    @Test
    public void  test2() {

        JdbcRealm jdbcRealm = new JdbcRealm();
        jdbcRealm.setDataSource(dataSource); //设置数据源
        jdbcRealm.setPermissionsLookupEnabled(true); //是否可以查看权限。默认为false。不打开，则无法查看权限

        //创建sql语句，使用自己的表、sql来验证
        String sql_auth = "SELECT `password` FROM test_user WHERE user_name = ?";
        jdbcRealm.setAuthenticationQuery(sql_auth);
        String sql_role = "SELECT role_name FROM test_user_role WHERE test_user_name = ?";
        jdbcRealm.setUserRolesQuery(sql_role);
        String sql_permisstion = "SELECT permission FROM test_role_permissions WHERE role_name = ?";
        jdbcRealm.setPermissionsQuery(sql_permisstion);

        //1.创建securityManager环境
        DefaultSecurityManager defaultSecurityManager = new DefaultSecurityManager();
        defaultSecurityManager.setRealm(jdbcRealm);

        //2.主体提交认证请求
        SecurityUtils.setSecurityManager(defaultSecurityManager);
        Subject subject = SecurityUtils.getSubject();

        UsernamePasswordToken token = new UsernamePasswordToken("xr","123456");//用户
        subject.login(token);
        System.out.println("isAuthenticated:" + subject.isAuthenticated());//已认证

        subject.checkRole("caiwu");

        //权限
        subject.checkPermission("user:update");//jdbcRealm.setPermissionsLookupEnabled(true);
    }


}

```

#### 3.自定义Realm

- 创建类CustomRealm

自定义Realm需要继承`AuthorizingRealm`。

```java
package com.example.springboot2shirostart;

import org.apache.shiro.authc.AuthenticationException;
import org.apache.shiro.authc.AuthenticationInfo;
import org.apache.shiro.authc.AuthenticationToken;
import org.apache.shiro.authc.SimpleAuthenticationInfo;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.authz.SimpleAuthorizationInfo;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;

import java.util.HashMap;
import java.util.HashSet;
import java.util.Map;
import java.util.Set;

public class CustomRealm extends AuthorizingRealm {

    final String realmName = this.getClass().getSimpleName(); //自定义realm名称

    Map<String,String> userMap = new HashMap<>(16);

    {
        userMap.put("zmt","123456");
        userMap.put("xr","654321");

        super.setName(realmName); //设置名称
    }


    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        String username = (String) principals.getPrimaryPrincipal();
        //从数据库或者缓存中获取角色数据。
        Set<String> roles = getRolesByUsername(username);
        Set<String> permissions = getPermissionByUsername(username);

        SimpleAuthorizationInfo authorizationInfo = new SimpleAuthorizationInfo();
        authorizationInfo.setStringPermissions(permissions); //设置权限
        authorizationInfo.setRoles(roles); //设置角色

        return authorizationInfo;
    }

    /**
     * 模拟通过用户名获取权限。
     * @param username
     * @return
     */
    private Set<String> getPermissionByUsername(String username) {
        Set<String> permissions = new HashSet<>();
        permissions.add("admin:delete");
        permissions.add("user:select");
        permissions.add("user:update");
        return permissions;
    }

    /**
     * 模拟获取角色。根据用户名获取角色。
     * @param username
     * @return
     */
    private Set<String> getRolesByUsername(String username) {
        Set<String> roles = new HashSet<>();
        roles.add("admin");
        roles.add("user");
        return roles;
    }

    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {

        //从主体传过来认证信息,获得用户名
        String username = (String) token.getPrincipal();
        System.out.println("username:" + username);

        //通过用户名到数据库中获取凭证
        String pwd = getPwdByUsername(username);
        if (pwd == null) {
            return null;
        }

        SimpleAuthenticationInfo authenticationInfo = new SimpleAuthenticationInfo(username,pwd,realmName);

        return authenticationInfo;
    }

    /**
     * 模拟获取数据库数据。根据用户名获取用户密码
     * @param username 用户名。
     * @return
     */
    private String getPwdByUsername(String username) {
        return userMap.get(username);
    }

}

```

- 测试：

```java
package com.example.springboot2shirostart;

import org.apache.shiro.SecurityUtils;
import org.apache.shiro.authc.UsernamePasswordToken;
import org.apache.shiro.mgt.DefaultSecurityManager;
import org.apache.shiro.subject.Subject;
import org.junit.Test;

/**
 * 简单认证
 */
public class CustomRealmTest {


    @Test
    public void  test1() {

        CustomRealm customRealm = new CustomRealm();//自定义Realm

        //1.创建securityManager环境
        DefaultSecurityManager defaultSecurityManager = new DefaultSecurityManager();
        defaultSecurityManager.setRealm(customRealm);

        //2.主体提交认证请求
        SecurityUtils.setSecurityManager(defaultSecurityManager);
        Subject subject = SecurityUtils.getSubject();

        UsernamePasswordToken token = new UsernamePasswordToken("zmt","123456");//用户
        subject.login(token);
        System.out.println("isAuthenticated:" + subject.isAuthenticated());//已认证

//        subject.checkRole("admin"); //admin通过，admin1报错
        subject.checkRoles("admin","user"); //拥有该两个角色

        //权限
        subject.checkPermission("user:select");//jdbcRealm.setPermissionsLookupEnabled(true);
//        subject.checkPermission("user:insert"); //报错，么有insert权限
        subject.checkPermission("user:update"); //有update权限
    }
}

```

### 五.Shiro加密

前面介绍的密码都是明文的，实际的密码在数据库中是加密的。    

下面在自定义的Realm中使用加密。  

- 代码：

```java
package com.example.springboot2shirostart;

import org.apache.shiro.authc.AuthenticationException;
import org.apache.shiro.authc.AuthenticationInfo;
import org.apache.shiro.authc.AuthenticationToken;
import org.apache.shiro.authc.SimpleAuthenticationInfo;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.authz.SimpleAuthorizationInfo;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;
import org.apache.shiro.util.ByteSource;

import java.util.HashMap;
import java.util.HashSet;
import java.util.Map;
import java.util.Set;

public class CustomRealm extends AuthorizingRealm {

    final String realmName = this.getClass().getSimpleName(); //自定义realm名称

    Map<String,String> userMap = new HashMap<>(16);

    {
//        userMap.put("zmt","123456");
//        userMap.put("zmt","e10adc3949ba59abbe56e057f20f883e"); //数据库的密码是密文，不加盐
        userMap.put("zmt","640a19b710290a9ff4d72e70cdd21913"); //md5加盐密码
        userMap.put("xr","654321");

        super.setName(realmName); //设置名称
    }


    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        String username = (String) principals.getPrimaryPrincipal();
        //从数据库或者缓存中获取角色数据。
        Set<String> roles = getRolesByUsername(username);
        Set<String> permissions = getPermissionByUsername(username);

        SimpleAuthorizationInfo authorizationInfo = new SimpleAuthorizationInfo();
        authorizationInfo.setStringPermissions(permissions); //设置权限
        authorizationInfo.setRoles(roles); //设置角色

        return authorizationInfo;
    }

    /**
     * 模拟通过用户名获取权限。
     * @param username
     * @return
     */
    private Set<String> getPermissionByUsername(String username) {
        Set<String> permissions = new HashSet<>();
        permissions.add("admin:delete");
        permissions.add("user:select");
        permissions.add("user:update");
        return permissions;
    }

    /**
     * 模拟获取角色。根据用户名获取角色。
     * @param username
     * @return
     */
    private Set<String> getRolesByUsername(String username) {
        Set<String> roles = new HashSet<>();
        roles.add("admin");
        roles.add("user");
        return roles;
    }

    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {

        //从主体传过来认证信息,获得用户名
        String username = (String) token.getPrincipal();
        System.out.println("username:" + username);

        //通过用户名到数据库中获取凭证
        String pwd = getPwdByUsername(username);
        if (pwd == null) {
            return null;
        }

        SimpleAuthenticationInfo authenticationInfo = new SimpleAuthenticationInfo(username,pwd,realmName);
        authenticationInfo.setCredentialsSalt(ByteSource.Util.bytes("aaa")); //密码加盐后，这里要加上这句

        return authenticationInfo;
    }

    /**
     * 模拟获取数据库数据。根据用户名获取用户密码
     * @param username 用户名。
     * @return
     */
    private String getPwdByUsername(String username) {
        return userMap.get(username);
    }


}

```

```java
package com.example.springboot2shirostart;

import org.apache.shiro.SecurityUtils;
import org.apache.shiro.authc.UsernamePasswordToken;
import org.apache.shiro.authc.credential.HashedCredentialsMatcher;
import org.apache.shiro.crypto.hash.Md5Hash;
import org.apache.shiro.mgt.DefaultSecurityManager;
import org.apache.shiro.subject.Subject;
import org.junit.Test;

/**
 * 简单认证
 */
public class CustomRealmTest {


    @Test
    public void  test1() {

        CustomRealm customRealm = new CustomRealm();//自定义Realm

        //1.创建securityManager环境
        DefaultSecurityManager defaultSecurityManager = new DefaultSecurityManager();
        defaultSecurityManager.setRealm(customRealm);

        //设置散列加密
        HashedCredentialsMatcher matcher = new HashedCredentialsMatcher();
        matcher.setHashAlgorithmName("md5");//设置加密方式
        matcher.setHashIterations(1); //设置加密次数
        customRealm.setCredentialsMatcher(matcher); //设置加密对象

        //2.主体提交认证请求
        SecurityUtils.setSecurityManager(defaultSecurityManager);
        Subject subject = SecurityUtils.getSubject();

        UsernamePasswordToken token = new UsernamePasswordToken("zmt","e10adc3949ba59abbe56e057f20f883e");//用户,密码MD5加密。明文：123456
        subject.login(token);
        System.out.println("isAuthenticated:" + subject.isAuthenticated());//已认证

//        subject.checkRole("admin"); //admin通过，admin1报错
        subject.checkRoles("admin","user"); //拥有该两个角色

        //权限
        subject.checkPermission("user:select");//jdbcRealm.setPermissionsLookupEnabled(true);
//        subject.checkPermission("user:insert"); //报错，么有insert权限
        subject.checkPermission("user:update"); //有update权限
    }

    @Test
    public void genPwd() {
//        Md5Hash md5Hash = new Md5Hash("123456"); //md5加密密码，不加盐
//        System.out.println("md5加密：" + md5Hash);

        Md5Hash md5Hash = new Md5Hash("e10adc3949ba59abbe56e057f20f883e","aaa"); //md5加密密码，加盐，密码更加难以识破，盐一般用随机数，这里写死
        System.out.println("md5加盐加密：" + md5Hash);
    }
}

```

### 六.shiro会话




