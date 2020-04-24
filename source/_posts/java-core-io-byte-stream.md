---
title: Java IO：第二节-字节流
date: 2019-09-29 09:47:37
categories: java-io
tags:
---

程序通过字节流去执行8位的字节的输出输入。所有的字节流的类都来自于`InputStream`和`OutputStream`，都继承这两个类。  

本节探索相关字节流的使用……



## 字节流使用

java io库中有很多的字节流的类。下面我们演示字节流是怎么工作的，这里我们用文件输入输出的字节流类来演示，`FileInputStream`和`FileOutputStream`。其他的字节流类，用法都类似，只是数据构造不同而已。

    public static void copyBytes() throws Exception {
        FileInputStream in = null;
        FileOutputStream out = null;

            try {
            in = new FileInputStream("xanadu.txt");
            out = new FileOutputStream("outagain.txt");
            int c;

            while ((c = in.read()) != -1) {
                out.write(c);
            }
        } finally {
            if (in != null) {
                in.close();
            }
            if (out != null) {
                out.close();
            }
        }
    }
    
上面方法，通过文件字节流，把文件内容从一个文件拷贝到另外一个文件。   

通过一个简单的循环，每次一个字节，把文件内容读入输入流，然后通过输出流写入到另外一个文件。如下图：

{% asset_img a-1.png %}    

### 关闭流

当你不再使用流的时候，一定要关闭流，避免严重的资源泄露。上面方法，通过`finally`,总是会关闭流，确保安全。

    finally {
        if (in != null) {
            in.close();
        }
        if (out != null) {
            out.close();
        }
    }

### 什么时候不用字节流

上面方法看起来像是一种通过的流操作方式，但是它实际代表的是一种低层次的I/O操作，应该避免。有用文件中包含的是字符，更恰当的方式应该是使用字符流。有很多适合操作更加复杂数据类型的流。但是字节流是最原始的流，用在操作原始数据类型上更为合适。

这里我们之所以讨论字节流，是因为其它操作更加复杂的类型的流都是建立在字节流之上的。

## `InputStream`和`OutputStream`

`InputStream`是字节输入流的最顶级抽象类，其实现两个接口。类图如下：

{% asset_img b-1.png %}   

`OutputStream`是字节输出流的最顶级抽象类，类图：

{% asset_img b-2.png %} 

## ByteArrayInputStream,ByteArrayOutputStream,FilterInputStream,FilterOutputStream

1.ByteArrayInputStream允许你从字节数组中读取字节流数据，代码如下：

    byte[] bytes = new byte[1024];
    
    //write data into byte array...
    
    InputStream input = new ByteArrayInputStream(bytes);
    
    //read first byte
    int data = input.read();
    while(data != -1) {
        //do something with data
    
        //read next byte
        data = input.read();
    }
    
类图：

{% asset_img c-1.png %}     

2.ByteArrayOutputStream允许你以数组的形式获取写入到该输出流中的数据，代码如下：

    ByteArrayOutputStream output = new ByteArrayOutputStream();
    
    output.write("This text is converted to bytes".getBytes("UTF-8"));
    
    byte[] bytes = output.toByteArray();

类图：

{% asset_img c-2.png %}  
        
3.FilterInputStream是实现自定义过滤输入流的基类，基本上它仅仅只是覆盖了InputStream中的所有方法。

就我自己而言，我没发现这个类明显的用途。除了构造函数取一个InputStream变量作为参数之外，我没看到FilterInputStream任何对InputStream新增或者修改的地方。如果你选择继承FilterInputStream实现自定义的类，同样也可以直接继承自InputStream从而避免额外的类层级结构。

4.FilterOutputStream，内容同FilterInputStream，不再赘述。