---
title: Java IO 缓存流
date: 2019-10-11 08:41:11
categories: java-io
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

4.关闭BufferedInputStream

确保关闭流，避免耗尽系统资源。关闭流，同时会刷出数据。

    BufferedInputStream bufferedInputStream = new BufferedInputStream(
                          new FileInputStream("c:\\data\\input-file.txt"));
    
    int data = bufferedInputStream.read();
    while(data != -1) {
      data = bufferedInputStream.read();
    }
    bufferedInputStream.close();
    
以上代码不能保证百分百关闭流，因为如果上面代码抛出异常，将无法执行`bufferedInputStream.close()`,此时，需要捕捉异常，保证不管何时，都能正确关闭流。下面利用`try-with-resources`保证总能准确关闭。    

    try(BufferedInputStream bufferedInputStream =
            new BufferedInputStream( new FileInputStream("c:\\data\\input-file.txt") ) ) {
    
        int data = bufferedInputStream.read();
        while(data != -1){
            data = bufferedInputStream.read();
        }
    }
    
一旦执行线程退出try块，就关闭BufferedInputStream。如果从try块内部引发了异常，则将捕获该异常，并关闭BufferedInputStream，然后重新引发该异常。因此，可以保证在try-with-resources块中使用BufferedInputStream时将其关闭。    

## Java BufferedOutputStream   

捕获缓冲区中写入`BufferedOutputStream`中的数据，批量的读入到底层的`OutputStream`中，提高IO的速度。

1.创建BufferedOutputStream

为`OutputStream`添加缓冲，只需要将其包装下即可，如下：

    OutputStream output = new BufferedOutputStream(
                          new FileOutputStream("c:\\data\\output-file.txt"));
                          
2.设置`BufferedOutputStream`缓冲区的大小

    int bufferSize = 8 * 1024;
    OutputStream output = new BufferedOutputStream(
                          new FileOutputStream("c:\\data\\output-file.txt"),
                              bufferSize
    );
    
设置的大小，最好是1024的倍数，这与硬盘中的大多数内置缓冲效果最佳。

3.如何选择最佳的缓冲区大小

缓冲区大小的设置，与io硬件设备息息相关。

设置不同的bufferSize实验，找出哪种缓冲区大小似乎可以在您的具体硬件上提供最佳性能。    

如果硬盘一次至少写入4kb，那么设置缓冲区小于4kb是不合理的，设置成6kb也是愚蠢的，这会造成硬盘碎片化，浪费存储空间。最好设置成4kb的倍数。

即使你的硬盘一次写入4kb，也不应该设置缓冲区大小为4kb，硬盘擅长按顺序写入数据，这意昧着它擅长写入相连的多个块。因此，设置缓冲区大小大于4kb比如16kb，32kb或者更大，性能更好。

要找到最佳的缓冲区大小，先要找到硬盘写入的块的大小，然后把缓冲区设置成块的倍数。最后实验不同倍数写入数据，看哪个倍数的大小写入速度最快，就用那个。

4.写入

使用`write()`方法写入数据到缓冲区。

    BufferedOutputStream bufferedOutputStream =
        new BufferedOutputStream(new FileOutputStream("c:\\data\\output-text.txt"));
    
    bufferedOutputStream.write(123);
    
上面例子，把`123`写入到给定的缓冲区。

5.以字节数组形式写入

由于Java BufferedOutputStream是OutputStream的子类，因此您也可以将字节数组写入BufferedOutputStream，而不是一次只写入一个字节。

    BufferedOutputStream bufferedOutputStream =
        new BufferedOutputStream(new FileOutputStream("c:\\data\\output-text.txt"));
    
    byte bytes =  new byte[]{1,2,3,4,5};
    
    outputStream.write(bytes);                              

6.刷新缓冲区flush()

当您将数据写入Java BufferedOutputStream时，数据会在内部缓存在字节缓冲区中，直到字节缓冲区已满为止，这时整个缓冲区都将写入底层的OutputStream中。

调用`flush()`方法刷新确保缓冲区的数据输出到硬盘等io设备。而不必等缓冲区满了，自动关闭再输出。

    OutputStream outputStream =
        new BufferedOutputStream(new FileOutputStream("c:\\data\\output-text.txt"));
    
    byte bytes =  new byte[]{1,2,3,4,5};
    
    outputStream.write(bytes);
    
    outputStream.flush()

7.关闭缓冲输出流BufferedOutputStream

使用缓冲区记得关闭，底层的输出流也会自动关闭。否则会极大浪费操作系统资源。

    BufferedOutputStream bufferedOutputStream = new BufferedOutputStream(
                          new FileOutputStream("c:\\data\\output-file.txt"));
    
    while(hasMoreData()) {
        int data = getMoreData();
        bufferedOutputStream.write(data);
    }
    bufferedOutputStream.close();        
    
java1.7以上使用方式：

放到try块里面，会自动会关闭资源。

    try( BufferedOutputStream bufferedOutputStream =
            new BufferedOutputStream(new FileOutputStream("c:\\data\\output-text.txt"))) {
    
        while(hasMoreData()) {
            int data = getMoreData();
            output.write(data);
        }
    }                                            