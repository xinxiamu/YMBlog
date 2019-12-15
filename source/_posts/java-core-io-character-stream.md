---
title: Java IO 字符流
date: 2019-09-29 15:36:38
categories: java-io
tags:
---

Java使用统一编码来存储字符，字符流能够根据本地字符集自动的转换格式。英语国家使用的是八位的ASCII超集。

程序中，使用字符流不再像字节流那么复杂，输入输出自动的根据本地字符集转换，国际化支持非常友好。

## 使用字符流

所有的字符流的类都继承自`Reader`和`Writer`,和字节流一样，下面我们演示文件字符流的使用,`FileReader`和`FileWriter`:

    @Test
    public void copyCharacters() throws IOException {
        FileReader inputStream = null;
        FileWriter outputStream = null;

        try {
            inputStream = new FileReader("xanadu.txt");
            outputStream = new FileWriter("characteroutput.txt");

            int c;
            while ((c = inputStream.read()) != -1) {
                outputStream.write(c);
            }
        } finally {
            if (inputStream != null) {
                inputStream.close();
            }
            if (outputStream != null) {
                outputStream.close();
            }
        }
    }
    
可以debug进去，查看到`c`变量的值，一个汉字，一个字母的`int`值。这里涉及到字符集以及字符编码的知识。

这里类似前面介绍的字节流，最大的不同，字节流用的是`FileInputStream`和`FileOutputStream`,字符流用的是`FileReader`和`FileWriter`。  

字节流的读取，字符流的读取，都用一个`int`变量来表示，来读取和写入。但是，字节流用的是`8 bits`的整型来表示，这里的字符流用的是`16 bits`的整型表示。  

## 字符流与字节流

字符流是字节流更上一层的包装，字符流实际使用的是字节流来处理底层的物流I/O的。如`FileReader`，继承`InputStreamReader`,并在该父类中进行字符和字节之间的转换。同样，`FileWriter`在父类`OutputStreamWriter`进行字符和字节的转换。

## 面向行的 I/O

字符I/O通常的可以比单个字符更大的单位类处理字符，通常可以以一行为单位来读取，以一行作为一个字符串来读取写入。行的终结符通常是("\r\n")，单回车("\r")或者单换行("\n")。下面例子：

        //逐行读取文本
        @Test
        public void copyCharactersLine() throws IOException {
            BufferedReader inputStream = null;
            PrintWriter outputStream = null;
    
            try {
                inputStream = new BufferedReader(new FileReader("xanadu.txt"));
                outputStream = new PrintWriter(new FileWriter("characteroutput.txt"));
    
                String l; //表示一行
                while ((l = inputStream.readLine()) != null) {
                    outputStream.write(l);
                }
            } finally {
                if (inputStream != null) {
                    inputStream.close();
                }
                if (outputStream != null) {
                    outputStream.close();
                }
            }
        }

## 包装字节流为字符流

把字节流转换成字符流，以字符形式读取。

1.当你有一个字节流`InputStream`,且你想以字符形式读取它，那么你可以把它包装到`InputStreamReader`中，如下构造：

    Reader reader = new InputStreamReader(inputStream);
    
其它的构造方式，可以查看`InputStreamReader`类的源码。    
    
2.同样的，字节输出流`OutputStream`也可以包装成字符输出流，如下：

    Writer writer = new OutputStreamWriter(outputStream);    
    
## 包装字符流

把字符流向上包装，以更大单位比如行字符串方式读取内容。如上面逐行读取。

    Reader reader = new BufferedReader(new FileReader(...));
    
    Writer writer = new BufferedWriter(new FileWriter(...));

    