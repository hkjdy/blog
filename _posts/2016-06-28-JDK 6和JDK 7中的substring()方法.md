---
date: 2016/6/28 星期二  20:08:34+0800
layout: post
title: JDK 6和JDK 7中的substring()方法
thread: 164
categories: java
tags:  substring
---

`substring(int beginIndex, int endIndex)`方法在`JDK 6`和`JDK7`中是不同的，了解二者之间的区别有助于更好的使用它们。简单起见，下文用`substring()`代替`substring(int beginIndex, int endIndex)`方法。

1.substring()实现什么功能？
------

`substring(int beginIndex, int endIndex)`方法返回字符串中索引位置从`beginIndex`到`endIndex-1`的子串。

```java
String x = "abcdef";
x = x.substring(1,3);
System.out.println(x);
```
输出:

```
bc
```

2.substring()被调用后发生了什么？
------

你可能知道，因为x是不可变的，把`x.substring(1,3)`的结果分配给`x`，`x`向下面这样指向一个新的完整字符串：
![](/assets/string-immutability1-650x303.jpeg)

然而，这张表并不能准确的说明堆(heap)中真实发生的事情。在`JDK 6`和`JDK 7`中`substring()`被调用后它们真实发生的事情是不同的。

3.JDK 6中的substring()
------

String是通过char数组来实现的。JDK 6中，String类包含3个域：`char value[]`， `int offset`，int count。分别用来存储真实的字符数组，数组的第一个索引位置，String中字符的长度。

调用substring()方法会创建一个新的字符串，但是字符串的值在堆中仍然指向相同的数组。两个字符串的区别是它们的`count`值和`value`值。

![](/assets/string-substring-jdk6-650x389.jpeg)

为了解释这个问题，下面的代码做过简化，而且只包含关键点。

```java
//JDK 6
String(int offset, int count, char value[]) {
	this.value = value;
	this.offset = offset;
	this.count = count;
}
 
public String substring(int beginIndex, int endIndex) {
	//check boundary
	return  new String(offset + beginIndex, endIndex - beginIndex, value);
}

```

4.JDK 6中substring()引发的问题
------
假如有一个很长很长的字符串，但是每次只需要使用substring()返回字符串很小的一部分。将会引起执行效率的问题，因为你维持了整个数组。JDK 6中通过以下方法解决这个问题，因为它真正的指向了子串。

```java
x = x.substring(x, y) + ""
```

5.JDK 7 中的substring()
------

下面是JDK 7中的改进。在JDK 7中，substring()方法实际上是在堆中创建了新的数组。

![](/assets/string-substring-jdk71-650x389.jpeg)

```
//JDK 7
public String(char value[], int offset, int count) {
	//check boundary
	this.value = Arrays.copyOfRange(value, offset, offset + count);
}
 
public String substring(int beginIndex, int endIndex) {
	//check boundary
	int subLen = endIndex - beginIndex;
	return new String(value, beginIndex, subLen);
}
```

英文原文：[The substring() Method in JDK 6 and JDK 7](http://www.programcreek.com/2013/09/the-substring-method-in-jdk-6-and-jdk-7/)