---
date: 2016/12/10 星期六  16:48:22+0800
layout: post
title: HeadFirst设计模式-单例模式
thread: 164
categories: 读书笔记
tags:  设计模式 单例模式
---

本章介绍的模式用来创建独一无二的，只能有一个实例的对象，称为单例模式。

## 为什么使用单例模式

有一些对象其实我们只需要一个，比方说：线程池、缓存、日志对象等。这些类对象只能有一个实例，如果创建多个实例会导致许多问题产生，例如：程序的行为异常，资源使用过量，或者是不一致的结果。

## 经典的单例模式实现

```java
public class Singleton {
	//利用一个静态变量来记录Singleton类的唯一实例
	private static Singleton uniqueInstance;
	
	//构造器声明为私有的，只有自Singleton类内才可以调用构造器
	private Singleton() {};
	
	//实例化对象，并返回这个实例
	public static Singleton getInstance() {
		//如果uniqueInstance是空的，表示还没有创建实例
		if (uniqueInstance == null) {
			uniqueInstance = new Singleton();
		}
		//如果uniqueInstance不是null，就表示之前已经创建过对象
		return uniqueInstance;
	}
}
```

Singleton 类没有公开的构造器，外部方法为了取得Singleton实例必须调用getInstance()静态方法。事实上Singleton实例是首次调用getInstance()方法才被创建出来的。

## 巧克力工厂

下面通过巧克力工厂系统的实现进一步了解单例模式。现代化的巧克力工厂具备计算机控制的巧克力锅炉。锅炉做的事就是把巧克力和牛奶融在一起，然后送到下一个阶段，以制造成巧克力棒。巧克力锅炉控制器的程序清单如下：

```java
public class ChocolateBoiler {
	private boolean empty;
	private boolean boiled;
	
	private static ChocolateBoiler uniqueInstance;
	
	private ChocolateBoiler() {
		empty = true;		//代码开始时，锅炉是空的
		boiled = false;
	}
	
	public static ChocolateBoiler getInstance() 
	{
		if (uniqueInstance == null) {
			uniqueInstance = new ChocolateBoiler();
		}
		return uniqueInstance;
	}
	
	public void fill() {
		if (isEmpty()) {	//往锅炉内填入原料时，锅炉必须是空的。
			empty = false;
			boiled = false;
		}
	}
	
	public void drain() {
		if (!isEmpty() && isBoiled()) {	//锅炉排出时，必须是满的，而且是煮过的
			empty = true;
		}
	}
	
	public void boil() {
		if(!isEmpty() && !isBoiled()) {	//煮混合物时，锅炉必须是满的，并且是没有煮过的。
			boiled = true;
		}
	}

	public boolean isEmpty() {
		return empty;
	}

	public void setEmpty(boolean empty) {
		this.empty = empty;
	}

	public boolean isBoiled() {
		return boiled;
	}

	public void setBoiled(boolean boiled) {
		this.boiled = boiled;
	}

}
```

现在我们使用单例模式完成了巧克力锅炉的设计，下面来看看单例模式的定义。

## 单例模式定义

**确保一个类只有一个实例，并提供一个全局访问点。**

我们正在把某个类设计成自己管理的一个单独实例，同时也避免其他类再自行产生实例。要想取得单例实例，通过单例类是唯一的途径。

## 单例模式类图

单例模式的类图可以说是所有模式的类图中最简单的，它的类图只有一个类。

![singleton](/assets/singleton/singleton1.png)


## 多线程带来的麻烦

为了充分利用CPU资源，我们打算使用多线程对ChocolateBoiler进行优化。来看看我们的测试代码：

```java
ChocolateBoiler instance = ChocolateBoiler.getInstance();
		
instance.fill();
instance.boil();
instance.drain();
```

在单线程环境中程序运行的很好，但是在多线程环境中问题就出现了。我们来模拟有两个线程同时调用getInstance()时可能会出现的情况。我们把每一个执行步骤都编上号，如下：

```java
public static ChocolateBoiler getInstance() 		//①
{
	if (uniqueInstance == null) {			//②
		uniqueInstance = new ChocolateBoiler();	//③
	}
	return uniqueInstance;				//④
}
```

线程可能会按照下列顺序执行：

![singleton](/assets/singleton/singleton2.png)

我们发现，当多个线程同时调用getInstance()时，可能会返回多个不同的对象。不符合 **一个类最多有一个实例** 的设计。下面的几种办法可以解决这个问题。

## 处理多线程

只需要把getInstance()变成同步(synchronized)方法，就可以轻松解决多线程问题：

```java
public static synchronized ChocolateBoiler getInstance() 
{
	if (uniqueInstance == null) {
		uniqueInstance = new ChocolateBoiler();
	}
	return uniqueInstance;
}
```

这样虽然可以解决多线程的问题，但是又带来了新的问题：同步会降低性能，因为只有第一次执行此方法时，才真正需要同步，之后每次调用这个方法，同步都是一种累赘。

## 改善多线程

我们有下面的选择可以改善多线程：

### 1.如果getInstance()的性能对应用程序不是很关键，就什么都别做。

同步getInstance()方法既简单又有效，如果getInstance()方法在程序中调用的不是很频繁，我们可以继续使用同步的getInstance()方法。

### 2.使用**急切**创建实例，而不用延迟实例化的做法。

如果应用程序总是创建并使用单例实例，或者**在创建和运行时方面的负担不太繁重**，你可能想要急切创建此单例，如下：

```java
public class Singleton {
	private static Singleton  uniqueInstance = new Singleton();
	
	private Singleton(){}
	
	public static Singleton getInstance() {
		return uniqueInstance;
	}
}
```

### 3.用**双重检查加锁**，在getInstance()中减少使用同步。

```java
public class Singleton {
	private volatile static Singleton uniqueInstance;
	
	private Singleton() {};
	
	public static Singleton getInstance() {
		if (uniqueInstance == null) {
			synchronized (Singleton.class) {
				if(uniqueInstance == null) {
					uniqueInstance = new Singleton();
				}
			}
		}
		return uniqueInstance;
	}
}
```

利用双重检查加锁，首先检查是否实例已经创建了，如果尚未创建，才进行同步，这样一来，只有第一次会同步，这正是我们想要的。

如果性能是你关心的重点，那么这个做法可以帮助你大大地减少getInstance()的时间耗费。

## 本章要点

1. 单例模式确保程序中一个类最多只有一个实例。

2. 单例模式也提供访问这个实例的全局点。

3. 在Java中实现单例模式需要私有的构造器，一个静态方法和一个静态变量。

4. 确定在性能和资源上的限制，然后小心地选择适当的方案来实现单例，以解决多线程的问题。
