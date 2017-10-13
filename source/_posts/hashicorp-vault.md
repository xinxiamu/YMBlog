---
title: HashiCorp Vault使用　
date: 2017-09-30 23:40:58
categories: security
tags: hashicorp-vault
---
## １. 简介
HashiCorp Vault是一个私密信息管理的工具。在应用开发中，特别是微服务开发中，用来更好的保护诸如数据库密码，api权限密码，第三方一些账号密码等，以避免在配置文件或者代码中明文硬编码，造成泄露。其在spring-cloud中也有很好的应用。
官网：https://www.vaultproject.io/

### 1.1 什么是私密信息
私密信息主要是一些需要保密的值或者键值对，很多时候会和敏感信息混淆。

这里举几个私密信息的例子：
> 1. 数据库登录信息
2. SSL证书
3. 云服务商的ACCESS KEY（比如AWS Cloud的IAM信息）
4. 其他加密用的密钥
5. API的认证信息

### 1.2 Vault的目标和特性
Vault的目标是成为私密信息的唯一来源，即一个集中化的管理工具。而私密信息的需求方可以程序化的获得所需的信息。对于私密信息，还应该有完善的审计和可视化方法，并且作为一个集中化的依赖，Vault自身必须是高可用的，对于云数据中心友好的安全架构。

Vault为了实现这些目标提供了以下特性：
> 1. 安全的私密信息存储
2. 动态的私密信息支持
3. 提供对于私密信息的更新，延长有效时间的功能
4. 高度灵活的权限控制
5. 多种客户端验证方式

## 2. Vault的使用

### 2.1 源码编译安装

1. 安装go环境，配置GOPATH。
查看以前配置记录，这里不做介绍。[golang](https://golang.org/)
2. 安装git环境
查看以前配置记录，这里不做介绍。[git](https://git-scm.com/)
3. 下载源码
> $ mkdir -p $GOPATH/src/github.com/hashicorp && cd $!
$ git clone https://github.com/hashicorp/vault.git
$ cd vault
4. 下载相关依赖包
> $ make bootstrap
5. 编译安装到./bin/下
> $ make dev
6. 验证安装是否成功
注意查看输出信息，确认vault在环境变量下。
> $ vault -v

《未完，待续……》





 
