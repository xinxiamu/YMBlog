---
title: Java IO 缓存流
date: 2019-10-11 08:41:11
categories: java
tags:
---

前两节我们看到的例子都没有使用缓存，使用的都是底层的I/O，每次的读写都是直接有操作系统来做的。这会导致很低效，每次的读写操作都是触发磁盘、网络或者其它的资源访问，频繁的访问这些资源是很昂贵的。

为了减少对这些底层资源的频繁访问，java设计了一套缓存API，它把读写数据都放到内存缓冲区里面，当缓存区为空的时候才调用系统底层读API取读取数据到内存，当缓冲区满的时候才调用底层系统输出API去刷新输出缓冲数据。这样，就大大的减少了调用操作系统接口操作硬件资源的次数。

## 把普通输出输出流包装成缓冲流

通过程序，我们可以把没有缓冲的输入输出流向上包装到缓冲输入输出流中。如下：

    inputStream = new BufferedReader(new FileReader("xanadu.txt"));
    outputStream = new BufferedWriter(new FileWriter("characteroutput.txt"));
    
有四个缓冲流用来包装普通的流，`BufferedInputStream` 和 `BufferedOutputStream`，用来包装字节流。 `BufferedReader`和`BufferedWriter`用来包装字符流。

## 刷新缓冲流

在关键的点写出缓冲区，不必等到每次缓冲区都满了再写出，这叫做刷新缓冲区。    

有些类支持自动缓冲，不必调用手动调用。比如`PrintWriter`类，每当调用`println` 或者 `format`，都会自动刷新一次缓冲区。

手动刷新缓冲区，直接调用方法`flush`，将会调用操作系统方法从缓冲区一次性把缓冲数据写入到对应媒介。该方法在所有的输出缓冲流中都有效。

## Java BufferedInputStream

包装输入字节流成缓冲字节流。一块一块的从缓冲区读取字节数据，不必逐个逐个调用操作系统API。

{%asset_img a-1.png%}

1.例如，把文件输入字节流包装成缓冲流：

    BufferedInputStream bufferedInputStream = new BufferedInputStream(
                          new FileInputStream("c:\\data\\input-file.txt"));
                          
`BufferedInputStream`会在内部构造一个字节数组，用来填充字节，然后调用底层方法`InputStream.read(byte[])`来读入。

2.设置缓冲区的长度

如下代码：

    int bufferSize = 8 * 1024; //8KB
        
    BufferedInputStream bufferedInputStream = new BufferedInputStream(
                          new FileInputStream("c:\\data\\input-file.txt"),
                          bufferSize); 

一般的，缓冲区的长度要是`1024 bytes`的倍数，整除。

3.设置最佳的缓冲区长度

很多时候，我们都有经验要问，缓冲区的长度到底要设置大小多少才是最佳性能的表现呢？        

实际上，这个问题，首先是要弄清楚你的硬盘缓冲区，每次刷新读取的大小的。如果你的硬盘缓冲区就4KB，那么，你设置缓冲流的大小小于4KB,或者大于4KB都是不明智的，小于4KB，那每次刷新，都浪费空间，大于，就要刷新两次。

所以，一般的，设置成和硬盘缓冲区一样的大小就好了。要通过各种长度测试，找出硬盘读取大小来确定。

4.mark() and reset()

`BufferInputStream`支持`mark()`和`reset()`方法。其继承自类`FilterInputStream`，类`FilterInputStream`是支持这两个方法的。   

并非所有InputStream子类都支持这些方法。       

可以调用方法`markSupported()`查看是否支持`mark()`和`reset()`。






                                                