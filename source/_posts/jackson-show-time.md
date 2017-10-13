---
title: jackson使用录
date: 2017-10-01 09:07:36
categories: json
tags: jackson
---
## 简介
Jackson库是一个“旨在为开发者提供更快，更正确，更轻量级，更符合人性思维” 的类库。Jackson为处理JSON格式提供了三种模型的处理方法。
1. 流式API或者增量解析/产生（ incremental parsing/generation）：读写JSON内容被作为离散的事件。

2. 树模型：提供一个可变内存树表示JSON文档。

3. 数据绑定（Data binding）：实现JSON与POJO（简单的Java对象（Plain Old Java Object））的转换。

一般的，我们更加关心json和javad对象的互相转换，这也是程序开发中最常用的。要使用jackson,要下载下面依赖包：
http://repo1.maven.org/maven2/com/fasterxml/jackson/core/

    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-core</artifactId>
        <version>2.9.1</version>
    </dependency>
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.9.1</version>
    </dependency>
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-annotations</artifactId>
        <version>2.9.1</version>
    </dependency>


## 1. Jackson Annotations介绍

