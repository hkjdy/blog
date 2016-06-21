---
date: 2016/6/18 星期六  15:48:05+0800
layout: post
title: Java中两种创建String的方式比较
thread: 164
categories: shell
tags:  Linux shell
---

Java中可以通过两种方式创建字符串：
```
String x = "abc";
String y = new String("abc");
```

两种创建字符串的方式有什么区别呢？

1. 双引号 vs. 构造函数
这个问题可以通过两个简单的代码示例来回答。

例1：
```
String a = "abcd";
String b = "abcd";
System.out.println(a == b);  // True
System.out.println(a.equals(b)); // True
```

`a==b` 结果为 `True` 是因为 `a` 和 `b` 在[method area](http://www.programcreek.com/2013/04/jvm-run-time-data-areas/)中引用的是相同的字符串文字（string literal）。引用的是相同的内存地址。
