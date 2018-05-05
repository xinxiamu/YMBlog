---
title: Spring cloud微服务：分布式配置中心-加密解密
date: 2018-04-21 22:22:30
categories: spring-cloud
tags: spring-cloud(配置中心加解密)
---

在spring cloud微服务开发过程中，很多的配置文件需要暴露给相关开发人员来维护。
但是，配置文件里面可能涉及到一些敏感的配置信息如数据库，第三方账号等。因此，为了安全，就非常有必要对这些数据加密了。以保护这些信息安全。


## 安装JCE

在使用Spring Cloud Config的加密解密功能时，有一个必要的前提需要我们注意。为了启用该功能，我们需要在配置中心的运行环境中安装不限长度的JCE版本（Unlimited Strength Java Cryptography Extension）。虽然，JCE功能在JRE中自带，但是默认使用的是有长度限制的版本。我们可以从Oracle的官方网站中下载到它，它是一个压缩包，解压后可以看到下面三个文件：

    README.txt
    local_policy.jar
    US_export_policy.jar

我们需要将local_policy.jar和US_export_policy.jar两个文件复制到`$JAVA_HOME/jre/lib/security`目录下，覆盖原来的默认内容。到这里，加密解密的准备工作就完成了。

JCE下载地址：[Java 8 JCE](http://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html)

## 查看相关端点

安装后，重启config-server。可以通过浏览器查看一些相关信息：

- /encrypt/status：查看加密功能状态的端点
- /key：查看密钥的端点
- /encrypt：对请求的body内容进行加密的端点
- /decrypt：对请求的body内容进行解密的端点

## 配置密钥

### 对称加解密

暂时不做介绍。推荐直接用下面非对称方式！

### 非对称加解密

使用非对称加解密具有更高安全性……

- 使用JDK工具keytool生成密钥对。

它的位置在： %JAVA_HOME%\bin\keytool.exe。

使用下面命令在当前文件夹下生成密钥对(默认有效期90天，这里设置成-validity 365天，一年)：

    keytool -genkeypair -alias config-server -keyalg RSA \ 
      -dname "CN=zhangmutian, OU=company, O=organization, L=city, ST=province, C=china" \
      -keypass 222222 \
      -keystore config-server.jks \
      -storepass 111111 \
      -validity 365 \

参考：
http://cloud.spring.io/spring-cloud-static/Edgware.SR3/multi/multi__spring_cloud_config_server.html#_creating_a_key_store_for_testing

## 配置bootstrap.yml

把生成的config-server.jks文件放在config-server项目的classpath中。并在bootstrap.yml中添加一下配置：

    encrypt:
      keyStore:
        location: classpath:/config-server.jks
        password: 11111
        alias: config-server
        secret: 222222

*注*：另外，以上配置信息也可以在环境变量中配置，它们对应的具体变量名如下：

    ENCRYPT_KEY_STORE_LOCATION
    ENCRYPT_KEY_STORE_ALIAS
    ENCRYPT_KEY_STORE_PASSWORD
    ENCRYPT_KEY_STORE_SECRET
    
重新启动项目，浏览：http://192.168.1.104:3331/encrypt/status  

    {
        "status": "OK"
    }
    
说明配置成功了。

## 对配置加密

先在命令窗口对具体配置明文进行加密,如：

对明文zmt加密（认证用户名为admin、密码为123456）

    mutian@mutian-ThinkPad-T440p:~$ curl -u admin:123456 http://192.168.1.104:ncrypt -d zmt
    AQB43S/okputI/v009zUuV/1XYmKSQROyYwWCWMC8phPeQa00/ABmS8QByz4ZWE57buwM1GIQ9lkmh8Yafgy6QUryq/XJk/oIck1zuN6M7IMepAFaJE4J8i4y5/LdH5h6gpfW06MeSiQbjg+393ztnDH37lWakfxEJ5yNtevXbV/LQC6u8bPvd/4riDHmgJYq8d7INJZKh4Y9TX+5a9a2YGivTuhn+qHruOylP43eMiK0EuUkmJF3B2zD6t8CWu5M84vnHjDVLFGmLuK3xfRpmG83ofl+86XjgdE+TlqcId+hRpfD28ELluU4Oc/N7ujNZAmKa2OtK0jve7oz27dQnrMDh5n6qkGAIcjNoeHLa7EgkP9XEargjGLkaXewHME56Q=  
    
对密文解密：

    mutian@mutian-ThinkPad-T440p:~$ curl -u admin:123456 http://192.168.1.104:3331/decrypt -d AQB43S/okputI/v009zUuV/1XYmKSQROyYwWCWMC8phPeQa00/ABmS8QByz4ZWE57buwM1GIQ9lkmh8Yafgy6QUryq/XJk/oIck1zuN6M7IMepAFaJE4J8i4y5/LdH5h6gpfW06MeSiQbjg+393ztnDH37lWakfxEJ5yNtevXbV/LQC6u8bPvd/4riDHmgJYq8d7INJZKh4Y9TX+5a9a2YGivTuhn+qHruOylP43eMiK0EuUkmJF3B2zD6t8CWu5M84vnHjDVLFGmLuK3xfRpmG83ofl+86XjgdE+TlqcId+hRpfD28ELluU4Oc/N7ujNZAmKa2OtK0jve7oz27dQnrMDh5n6qkGAIcjNoeHLa7EgkP9XEargjGLkaXewHME56Q=
    zmt   
    
在配置文件中配置密文：

通过上面用curl命令请求把明文加密后，然后如下添加到配置文件中。`{cipher}`代表是密文，需要解密。

    api:
      password: '{cipher}AQB0sf3nKuMq6wmRGs1CVDs3Oq+gdkfhX7F4M5txKI0CUpezKl02GI1mWmY4e6Ch/tI0UP9KRLv5VADrF8qESSPrZjD+uQR+op/N1hEZmKOMS/BpgipudiskeuifHPk2ffscN6pJns4VrfRwW3Io9yyOJ0/mAQxD46IcppraE2Z4gwplLvRU0U7pLB2mxpBqhi24ZKUW3MHRRD5rF4AMyXQw9SEyfyXYWpBGxSgMGfeV/TU4d4DVSYy8Y7Ji0Rf41m/59V24bjjYaJL2B77+WLyKlGHlV/hfrCOcz45NgqS00TGjNfieO1DlWHZi/YvYN4UUF0InRFI2gnGzWumEnJSYhHWqO3hdVr9mO+BI8DskngMGapYQrJVc7Pdpo27h3Io='

调用显示：

    @Value("${api.password}")
    private String apiPwd;

    @Override
    public VTestResp test3(@SensitiveFormat String name) {
        VTestResp testResp = new VTestResp();
        testResp.setName(name + ">>>>" + apiPwd);
        return testResp;
    }
    
    结果：
    {
        "name": "abc>>>>ymu123456",
        "sex": 0
    }
    
## 对特殊字符加密问题

参考：http://blog.didispace.com/spring-cloud-config-sp-char-encryp         