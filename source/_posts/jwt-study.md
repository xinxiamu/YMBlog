---
title: jwt初识
date: 2018-03-08 16:47:23
categories: jwt
tags:
---

JSON Web Token（JWT）是目前比较流行的跨域身份验证的解决方案。也是系统间信息交互的好助手。下面介绍其机制和原理，以及使用！

## jwt是什么

JSON Web Token (JWT)是一个开放标准(RFC 7519)，它定义了一种紧凑的、自包含的方式，用于作为JSON对象在各方之间安全地传输信息。该信息可以被验证和信任，因为它是数字签名的。

## 什么时候使用jwt

- Authentication（认证）、Authorization（授权）: 

这是jwt使用的最广的场景。一旦用户登录后，返回jwt令牌，后续客户端携带该令牌作为身份访问授权的api资源。可以使用jwt来做单点登录，开销很小，并且可以轻松的跨域。

- Information Exchange (信息交换) :

在各方系统之间安全的传递消息，jwt无疑是一种很好的方式。因为jwt可以被签名，例如用公钥/私钥对，可以确定发送人就是所指定的那个人。同时，还可以验证内容有没有被篡改。    

- 不依赖session机制： 

以往通过seesion会话机制来做验证，但是如果session保存在内存，就无法做分布式系统，因为分布式环境中要求请求无状态。即使把session持久化到如redis，关系数据库或者文件系统中能解决无状态问题，但是这个对业务侵入大，且持久化过程失败的话，整个验证过程就失败，系统开销也大。jwt直接把数据存储在客户端，每次请求携带上就可以认证，简单且开销小。

## JSON Web Token的结构

{% asset_img a-1.png %}

jwt要三部分组成，它们之间用圆点（.）隔开。三部分分别是：  

- Header
- Payload
- Signature

所以，一个jwt看起来应该是这个样子：

xxxxxxxxxx.yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy.zzzzzzzzzzzz

### Header

header由两部分组成：类型（“JWT”）和算法名称（比如：HMAC SHA256或者RSA等等）。

像下面：    

    {
        "alg": "HS256",
        "typ": "JWT"
    }

alg表示使用的签名算法，默认为HMAC SHA256（写为HS256）；typ表示令牌的类型，JWT令牌统一写为JWT。

最后，要用Base64URL算法对上面json对象编码转为字符串作为JWT的第一部分。   

### Payload

jwt的有效载体payload,声明关于实体（通常指用户）和其它数据，声明有三种类型：registered, public 和 private。    

- Registered claims :   

预定义的声明，不是强制的，但是推荐。指定7个默认的字段供选择： 

1.iss：发行人

2.exp：到期时间

3.sub：主题

4.aud：用户

5.nbf：在此之前不可用

6.iat：发布时间

7.jti：JWT ID用于标识该JW

- Public claims : 

公开的，可以随意定义。

- Private claims : 

用于在同意使用它们的各方之间共享信息，并且不是注册的或公开的声明。

下面是个例子：

    {
        "sub": "1234567890",
        "name": "chongchong",
        "admin": true
    }
    
对payload进行Base64URL编码就得到JWT的第二部分。

_注意：_  不要在JWT的payload或header中放置敏感信息，除非它们是加密的。使用https或者加密算法才可以添加敏感信息，否则有信息泄露可能。

### Signature签名哈希

为了得到签名部分，你必须有编码过的header、编码过的payload、一个秘钥，签名算法是header中指定的那个，然对它们签名即可。

例如： 

    HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret) 
    
签名是用于验证消息在传递过程中有没有被更改，并且，对于使用私钥签名的token，它还可以验证JWT的发送方是否为它所称的发送方。secret必须保持在服务器，不可泄露。    

### Base64URL算法

如前所述，JWT头和有效载荷序列化的算法都用到了Base64URL。该算法和常见Base64算法类似，稍有差别。

作为令牌的JWT可以放在URL中（例如api.example/?token=xxx）。 Base64中用的三个字符是"+"，"/"和"="，由于在URL中有特殊含义，因此Base64URL中对他们做了替换："="去掉，"+"用"-"替换，"/"用"_"替换，这就是Base64URL算法，很简单吧。

## jwt工作机制

认证的时候，当用户用他们的凭证登录成功后，服务端将会按照上面方式生成一个jwt返回给客户端。此后，该token就是用户的凭证了，后面每次请求都带上该token。你必须要非常小心的保护该token，在客户端保存令牌的时间不能超过令牌的有效时间。

每次客户端请求受保护的路由或者资源，都得带上该token，一般的，放在Authorization header中，用Bearer schema。

header应该看起来是这样的：    

    Authorization: Bearer <token>
    
服务端会在过滤器或者拦截器中，拦截每次请求，并检查Authorization header中的JWT是否有效。如果有效，则可以访问相关资源，否则返回token失效，此时客户端应该要重新登录获取最新的token。   

如果jwt中包含足够的必要数据，那么可以减少查询数据库的次数。尽管不一定必要。例如，保存用户`username`到jwt中，而不仅仅保存`id`,下次服务端接到请求，就可以直接使用`username`，而不必根据`id`再查一遍。

如果token是在授权头（Authorization header）中发送的，那么跨源资源共享(CORS)将不会成为问题，因为它不使用cookie。

注意：

1.每一次请求都需要token。    
2.Token应该放在请求header中。   
3.我们还需要将服务器设置为接受来自所有域的请求，用Access-Control-Allow-Origin: *。  

## JWT与OAuth的区别

1.OAuth2是一种授权框架 ，JWT是一种认证协议。    
2.无论使用哪种方式切记用HTTPS来保证数据的安全性。    
3.OAuth2用在使用第三方账号登录的情况(比如使用weibo, qq, github登录某个app)，而JWT是用在前后端分离, 需要简单的对后台API进行保护时使用。 

## 基于Token的身份认证 与 基于服务器的身份认证

## 总结   

1、JWT默认不加密，但可以加密。生成原始令牌后，可以使用改令牌再次对其进行加密。

2、当JWT未加密方法是，一些私密数据无法通过JWT传输。

3、JWT不仅可用于认证，还可用于信息交换。善用JWT有助于减少服务器请求数据库的次数。

4、JWT的最大缺点是服务器不保存会话状态，所以在使用期间不可能取消令牌或更改令牌的权限。也就是说，一旦JWT签发，在有效期内将会一直有效。

5、JWT本身包含认证信息，因此一旦信息泄露，任何人都可以获得令牌的所有权限。为了减少盗用，JWT的有效期不宜设置太长。对于某些重要操作，用户在使用时应该每次都进行进行身份验证。

6、为了减少盗用和窃取，JWT不建议使用HTTP协议来传输代码，而是使用加密的HTTPS协议进行传输。