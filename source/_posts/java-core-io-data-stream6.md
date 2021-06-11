---
title: Java IO：第六节-Data Stream
date: 2020-04-24 10:31:30
categories: java-io
tags:
---

数据流是支持原始的数据类型(boolean, char, byte, short, int, long, float, and double)以及字符串String类型的二进制I/O。所有的数据流都实现接口DataInputStream或者DataOutputStream。本节重点介绍这些接口的最广泛使用的实现，即DataInputStream和DataOutputStream。

{%asset_img %}

定义常量：
```text
static final String dataFile = "invoicedata";

static final double[] prices = { 19.99, 9.99, 15.99, 3.99, 4.99 };
static final int[] units = { 12, 8, 13, 29, 50 };
static final String[] descs = {
    "Java T-shirt",
    "Java Mug",
    "Duke Juggling Dolls",
    "Java Pin",
    "Java Key Chain"
};
```

