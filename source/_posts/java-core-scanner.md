---
title: Java IO Scanner And Formatting
date: 2019-11-11 09:34:53
categories: java-io
tags:
---

Scanner类型的对象可用于将格式化的输入分解为令牌，并根据其数据类型转换单个令牌。 

把输入流中的数据分解，找到对应的数据类型数据。比如在一个文本流中找出所有的数字类型等。

## Scanner

#### 将输入流分解

默认情况下，扫描仪使用空格分隔令牌（理解为分隔标识）。 （空格字符包括空格，制表符和行终止符。

    public class ScanXan {
        public static void main(String[] args) throws IOException {
    
            Scanner s = null;
    
            try {
                s = new Scanner(new BufferedReader(new FileReader("xanadu.txt")));
    
                while (s.hasNext()) {
                    System.out.println(s.next());
                }
            } finally {
                if (s != null) {
                    s.close();
                }
            }
        }
    }
    
文本`xanadu.txt`的内容：

    热烈 庆祝中国成立70周年100！！！
    
    We warmly celebrate the 70th anniversary of the founding of China!!!
    
输出：

    热烈
    庆祝中国成立70周年100！！！
    We
    warmly
    celebrate
    the
    70th
    anniversary
    of
    the
    founding
    of
    China!!!

可以看到，以空格为一个分隔符（令牌）。     

请注意，ScanXan在完成扫描程序对象时会调用Scanner的close方法。即使扫描程序不是流，您也需要关闭它以表明已完成其基础流。

当然，你可以使用不同的分隔符来分隔流，调用`useDelimiter()`方法来实现，传入一个正则表达式。

如：`s.useDelimiter(",\\s*")`,逗号分隔。

#### 翻译当个分隔值（tokens）

ScanXan示例将所有输入标记视为简单的String值。扫描程序还支持所有Java语言原始类型（char除外）以及BigInteger和BigDecimal的令牌。

此外，数值可以使用数千个分隔符。
因此，在美国语言环境中，扫描程序会正确读取代表整数值的字符串“ 32,767”。

     public static void main(String[] args) throws IOException {
    
            Scanner s = null;
            double sum = 0;
    
            try {
                s = new Scanner(new BufferedReader(new FileReader("usnumbers.txt")));
                s.useLocale(Locale.US);
    
                while (s.hasNext()) {
                    if (s.hasNextDouble()) {
                        sum += s.nextDouble();
                    } else {
                        s.next();
                    }   
                }
            } finally {
                s.close();
            }
    
            System.out.println(sum);
     }

以上代码，会把分隔的数字类型读取出来，并累加计算。

## Formatter



