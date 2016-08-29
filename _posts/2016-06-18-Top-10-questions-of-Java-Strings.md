---
date: 2016/06/18 星期六  16:23:53+0800
layout: post
title: Top 10 questions of Java Strings【转】
thread: 164
categories: Java
tags:  string
---


[原文地址](http://www.programcreek.com/2013/09/top-10-faqs-of-java-strings/)

------

The following are top 10 frequently asked questions about Java Strings.  

1.How to compare strings? Use "==" or use equals()?
------

In brief, "==" tests if references are equal and equals() tests if values are equal. Unless you want to check if two strings are the same object, you should always use equals().

It would be better if you know the concept of [string interning](http://www.programcreek.com/2013/04/why-string-is-immutable-in-java/).

2.Why is char[] preferred over String for security sensitive information?
------

[Strings are immutable](http://www.programcreek.com/2009/02/diagram-to-show-java-strings-immutability/), which means once they are created, they will stay unchanged until Garbage Collector kicks in. With an array, you can explicitly change its elements. In this way, security sensitive information(e.g. password) will not be present anywhere in the system.

3.Can we use string for switch statement?
------

Yes to version 7. From [JDK 7](http://openjdk.java.net/projects/jdk7/features/), we can use string as switch condition. Before version 6, we can not use string as switch condition.

```java
// java 7 only!
switch (str.toLowerCase()) {
      case "a":
           value = 1;
           break;
      case "b":
           value = 2;
           break;
}
```

4.How to convert string to int?
------

```java
int n = Integer.parseInt("10");
Simple, but so frequently used and sometimes ignored.
```

5.How to split a string with white space characters?
------

```java
String[] strArray = aString.split("\\s+");
```

6.What substring() method really does?
------

In JDK 6, the substring() method gives a window to an array of chars which represents the existing String, but do not create a new one. To create a new string represented by a new char array, you can do add an empty string like the following:

```java
str.substring(m, n) + ""
```

This will create a new char array that represents the new string. The above approach sometimes can make your code faster, because Garbage Collector can collect the unused large string and keep only the sub string.

In Oracle JDK 7, substring() creates a new char array, not uses the existing one. Check out the diagram for showing [substring() difference between JDK 6 and JDK 7](http://www.programcreek.com/2013/09/the-substring-method-in-jdk-6-and-jdk-7/).

7.String vs StringBuilder vs StringBuffer
------

String vs StringBuilder: StringBuilder is mutable, which means you can modify it after its creation.
StringBuilder vs StringBuffer: StringBuffer is synchronized, which means it is thread-safe but slower than StringBuilder.

8.How to repeat a string?
------

In Python, we can just multiply a number to repeat a string. In Java, we can use the repeat() method of StringUtils from Apache Commons Lang package.

```java
String str = "abcd";
String repeated = StringUtils.repeat(str,3);
//abcdabcdabcd
```

9.How to convert string to date?
------

```java
String str = "Sep 17, 2013";
Date date = new SimpleDateFormat("MMMM d, yy", Locale.ENGLISH).parse(str);
System.out.println(date);
//Tue Sep 17 00:00:00 EDT 2013
```

10.How to count # of occurrences of a character in a string?
------
Use StringUtils from apache commons lang.

```java
int n = StringUtils.countMatches("11112222", "1");
System.out.println(n);
```

One more
------

Do you know [How to detect if a string contains only uppercase letter](http://www.programcreek.com/2011/04/a-method-to-detect-if-string-contains-1-uppercase-letter-in-java/)?
