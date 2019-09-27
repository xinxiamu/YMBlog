---
title: Java IO 概述
date: 2019-09-27 09:40:26
categories: java
tags:
---

Java IO是一组API用来读取和写入数据(input and output)。大多数应用程序都需要处理一些输入，并在输入基础上输出一些数据。例如，从文件读取文件内容到内存，经过程序处理然后通过网络传输，或者反过来。       
该组API在`java.io`下，当你浏览该包时，这些类都有什么作用呢，该如何选择呢，下面我们来不断探索。

## Java.io 包的范围

java.io 包并没有涵盖所有输入输出类型。例如，并不包含GUI或者网页上的输入输出，这些输入和输出在其它地方都涉及，比如Swing工程中的JFC (Java Foundation Classes) 类，或者J2EE里的Servlet和HTTP包。       

Java.io 包主要涉及文件，网络数据流，内存缓冲等的输入输出。

## 输入和输出 – 数据源和目标媒介

术语`输入`,`输出`有时候会让人感到疑惑，实际上其描述的是数据在程序和媒介之间的一个流向问题。简单理解：以应用程序为中心（此时应用程序已经加载到内存形成一系列的指令集，等待执行），输入流可以理解成应用程序从外部媒介如文件读取数据内容到物理内存中，输出流可以理解成应用程序把内存中处理后的数据读出到外部媒介如文件。

Java的IO包主要关注的是从原始数据源的读取以及输出原始数据到目标媒介。以下是最典型的数据源和目标媒介：

- 文件
- 管道
- 网络连接
- 内存缓存 (e.g. arrays)
- System.in, System.out, System.error(注：Java标准输入、输出、错误输出)

下面图形象描述了程序从数据源读取数据到目标媒介：

{% asset_img a-1.png %}

## I/O 流

在Java IO中，流是一个核心的概念。流从概念上来说是一个连续的数据流。你既可以从流中读取数据，也可以往流中写数据。流与数据源或者数据流向的媒介相关联。在Java IO中流既可以是字节流(以字节为单位进行读写)，也可以是字符流(以字符为单位进行读写)。

一个I/O流代表一个输入源或者一个输出媒介，一个流可以代表各种不同的源和媒介，比如磁盘中的文件、各种设备、其他的程序、内存等。

流支持各种数据类型，包括简单的字节类型，原始数据类型，本地化的字符类型甚至更高级的类型如对象。一些流只是传递数据，然后进行各种转换。

一个流就是一系列的数据，应用程序利用输入流从数据源中读取这些数据到程序内存中：

{% asset_img b-1.png %}

程序利用输出类把内存数据写入到媒介如文件系统中：

{% asset_img b-2.png %}

## InputStream, OutputStream, Reader 和Writer类

程序从数据源读取数据需要依靠`inputStream`或者`Reader`，程序写数据到媒介中时候需要依赖`outputStream`或者`Writer`。下面图形象描述：

{% asset_img a-2.png %}

InputStream和Reader与数据源相关联，OutputStream和writer与目标媒介相关联。

## Java IO的用途和特征

Java IO包中有很多`InputStream`, `OutputStream`, `Reader` and `Writer`的子类，这些子类都具有某些自己的用途，总结如下：

- 文件访问
- 网络访问
- 内存缓存访问
- 线程内部通信（管道）
- 缓冲
- 过滤
- 解析
- 读写文本 (Readers / Writers)
- 读写基本类型数据 (long, int etc.)
- 读写对象

当我通读`java.io`包下相关类后，很容易就知道这些类的特性和其使用场景。

## Java IO类概述表

已经讨论了数据源、目标媒介、输入、输出和各类不同用途的Java IO类，接下来是一张通过输入、输出、基于字节或者字符、以及其他比如缓冲、解析之类的特定用途划分的大部分Java IO类的表格。

{% asset_img a-3.png %}

参考：
- https://docs.oracle.com/javase/tutorial/essential/io
- http://tutorials.jenkov.com/java-io
- http://commons.apache.org/proper/commons-io/
