---
date: 2016/12/2 星期五  12:23:53+0800
layout: post
title: HeadFirst设计模式-装饰者模式
thread: 164
categories: 读书笔记
tags:  设计模式 装饰者模式
---

本篇即将探讨典型的继承滥用问题，我们将学到如何使用对象组合的方式，做到在运行时装饰类。

## 定义

**动态地将责任附加到对象上。若要扩展功能，装饰者提供了比继承更有弹性的替代方案。**

## 星巴兹咖啡

首先，我们先看一个现有的咖啡订单系统的实现。

![coffee](/assets/coffee/coffee1.png)

Beverage 是一个抽象类，店内所提供的饮料都必须继承自此类。此抽象类有一个抽象的cost()方法，子类必须定义自己的实现。

购买咖啡时，也可以要求在其中加入各种调料，例如：Milk、Soy、Mocha或Whip。星巴兹会根据所加入的调料收取不同的费用。所以订单系统必须考虑到这些调料部分。

我们的第一个尝试是将每一种可能的调料组合都实现一个子类，结果就是我们实现了类似下面这样的一些类：

HouseBlendWithSteamedMilk、HouseBlendWithMocha、HouseBlendWithWhipandMocha ...

我们发现需要实现几十种类，给维护造成了很大的麻烦，如果调料的价格发生变化或者需要添加新的调料我们将难以维护。

这种实现被确认无法实现我们的需求，接下来我们换一种思路：利用实例变量和继承来看看能否达到我们的目的。程序清单如下：

`Beverage` 超类

```java
public class Beverage {
	//各种调料的布尔值
	private boolean milk;
	private boolean soy;
	private boolean mocha;
	private boolean whip;
	
	//各种调料的价格
	private double milkCost;
	private double soyCost;
	private double mochaCost;
	private double whipCost;
	
	public double cost(){
		double condimentCost = 0.0;

		if(this.hasMilk()) {
			condimentCost += milkCost;
		}
		
		if(this.hasSoy()) {
			condimentCost += soyCost;
		}
		
		if(this.hasMocha()) {
			condimentCost += mochaCost;
		}
		
		if(this.hasWhip()) {
			condimentCost += whipCost;
		}
		
		return condimentCost;
	}

	public boolean hasMilk() {
		return milk;
	}

	public void setMilk(boolean milk) {
		this.milk = milk;
	}

	public boolean hasSoy() {
		return soy;
	}

	public void setSoy(boolean soy) {
		this.soy = soy;
	}

	public boolean hasMocha() {
		return mocha;
	}

	public void setMocha(boolean mocha) {
		this.mocha = mocha;
	}

	public boolean hasWhip() {
		return whip;
	}

	public void setWhip(boolean whip) {
		this.whip = whip;
	}	
}

```

加入子类 `DarkRoast`，每个子类代表菜单上的一种饮料。

```java
public class DarkRoast extends Beverage {
	
	public DarkRoast() {}

	@Override
	public double cost() {
		return 1.99 + super.cost();
	}
}
```

修改以后的方案大幅减少的子类的数量，每一种饮料只需要实现一个子类就可以了。但是同时我们又遇到了一些问题：

1. 如果调料价钱发生变化，我们就需要更改现有代码。

2. 一旦出现新的调料，我们就需要加上新的方法，并改变超类中的cost()方法。

3. 以后可能会开发出新饮料。对这些饮料而言（例如：冰茶），某些调料可能并不适合，但是在这个设计方案中，Tea（茶）子类仍将继承那些不合适的方法。

4. 无法实现顾客想要双倍摩卡咖啡的要求。（加双份摩卡）

接下来，我们来介绍如何通过**装饰者模式**解决以上问题。

我们的设计思路是以饮料为主体，然后再用调料来装饰饮料。比方说，如果顾客想要摩卡和奶泡深焙咖啡，那么，要做的是：

1. 拿一个深焙(DarkRoast)咖啡对象。

2. 以摩卡对象装饰它。

3. 以奶泡对象装饰它。

4. 调用cost()方法，并依赖委托将调料的价钱加上去。

### 以装饰者构造饮料订单

1. 以DarkRoast对象开始

![coffee](/assets/coffee/coffee2.png)

2. 顾客想要摩卡(Mocha)，所以建立一个Mocha对象，并用它将DarkRoast对象包(wrap)起来。

![coffee](/assets/coffee/coffee3.png)

3. 顾客想要奶泡(Whip)，所以需要建立一个Whip装饰者，并用它将Mocha对象包起来。别忘了，DarkRoast继承自Beverage，且有一个cost()方法，用来计算饮料价钱。

![coffee](/assets/coffee/coffee4.png)

4. 现在，该是为顾客算钱的时候了。通过调用最外圈装饰者(Whip)的cost()就可以办得到。Whip的cost()会先委托它装饰的对象（也就是Mocha）计算出价钱，然后再加上奶泡的价钱。

![coffee](/assets/coffee/coffee5.png)

### 程序清单

#### Beverage超类

```java
public abstract class Beverage {
	public abstract double cost();
}
```

### CondimentDecorator（调料）抽象类

```java
public abstract class CondimentDecorator extends Beverage {

	@Override
	public double cost() {
		return 0;
	}
}
```

### 饮料代码

这里只实现两种饮料Espresso(价钱 $1.99)和HouseBlend(价钱 $0.89)。

```java
public class Espresso extends Beverage {

	@Override
	public double cost() {
		return 1.99;
	}

}

```

```java
public class HouseBlend extends Beverage {

	@Override
	public double cost() {
		return 0.89;
	}

}

```

### 调料代码

这里分别实现了Mocha(价钱 $0.20)、Soy(价钱 $0.30)、Whip(价钱 $0.40)调料类。

```java
public class Mocha extends CondimentDecorator {
	private Beverage beverage;
	
	public Mocha(Beverage beverage) {
		this.beverage = beverage;
	}

	@Override
	public double cost() {
		return 0.20 + beverage.cost();
	}
	
}

```

```java
public class Soy extends CondimentDecorator {
	private Beverage beverage;
	
	public Soy(Beverage beverage) {
		this.beverage = beverage;
	}

	@Override
	public double cost() {
		return 0.30 + beverage.cost();
	}
}

```

```java
public class Whip extends CondimentDecorator {
	private Beverage beverage;
	
	public Whip(Beverage beverage) {
		this.beverage = beverage;
	}
	
	@Override
	public double cost() {
		return 0.40 + beverage.cost();
	}
}

```

### 供应咖啡

这是用来下订单的一些测试代码

```java
public class StarbuzzCoffee {

	public static void main(String[] args) {
		Beverage beverage = new Espresso();
		//打印一杯Espresso，不加任何调料的价钱
		System.out.println("espresso: $" + beverage.cost());
		
		beverage = new Mocha(beverage);
		//加一份摩卡的价钱
		System.out.println("espresso: $" + beverage.cost());
		
		//加两份摩卡的价钱
		beverage = new Mocha(beverage);
		System.out.println("espresso: $" + beverage.cost());
		
		beverage = new Whip(beverage);	
		//一杯Espresso，加两份摩卡，一份Whip的价钱
		System.out.println("espresso: $" + beverage.cost());		
	}
}

```

### 打印结果

```
espresso: $1.99		//不加任何调料		$1.99
espresso: $2.19		//一份摩卡			$1.99+0.20
espresso: $2.39		//两份摩卡			$1.99+0.20+0.20
espresso: $2.79		//两份摩卡+一份奶泡	$1.99+0.20+0.20+0.40
```

## 总结

1. 装饰者和被装饰者有相同的超类型。

2. 你可以用一个或多个装饰者包装一个对象。

3. 既然装饰者和被装饰者对象有相同的超类型，所以在任何需要原始对象（被包装）的场合，可以用装饰过的对象代替它。

4. 装饰者可以在所委托被装饰者的行为之前/后，加上自己的行为，以达到特定的目的。

5. 对象可以在任何时候被装饰，所以可以在运行时动态地、不限量地用你喜欢的装饰者来装饰对象。

装饰者模式杰设计注入弹性，但是，装饰者模式会在设计中加入大量的小类，这偶尔会导致别人不容易理解这种设计方式。所以在使用装饰者时，必须要小心谨慎。后面介绍的工厂模式和生成器模式对解决这个问题会有所帮助。
