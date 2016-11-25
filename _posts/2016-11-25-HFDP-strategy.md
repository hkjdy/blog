---
date: 2016/11/25 星期五  17:18:08+0800
layout: post
title: HeadFirst设计模式-策略模式
thread: 164
categories: 读书笔记
tags:  设计模式 策略模式
---

本节，将学习为何（以及如何）利用其他开发人员的经验与智慧。我们会看到设计模式的使用与优点。

## 简单的模拟鸭子游戏：SimUDuck

### 老板的需求

SimUDuck游戏中会出现各种鸭子，可以游泳，可以呱呱叫。

### 实现

我们设计一个鸭子超类，并让各种鸭子继承此超类，程序清单如下：

```java
public abstract class Duck {
	public void quack() {}
	public void swim() {}
	//因为每一种鸭子的外观都不同，所以此方法是抽象的
	public abstract void display();	
}
```

绿头鸭子类

```java
public class MallardDuck extends Duck {

	@Override
	public void display() {
		//外观是绿头
	}

}
```
红头鸭子类

```java
public class RedheadDuck extends Duck {

	@Override
	public void display() {
		// 外观是红头
	}
}
```

鸭子只会游泳和呱呱叫还不够，我们还要让鸭子能飞，很简单，只需要在鸭子的超类中添加一个会飞的方法就搞定了，这样所有继承自此超类的各种鸭子就都会飞了。如下程序清单所示：

```java
public abstract class Duck {
	public void quack() {}
	public void swim() {}
	public void fly() {}	//新添加上飞行的方法
	public abstract void display();
}
```

接下我们要实现一个橡皮鸭类（RubberDuck）同样继承自鸭子超类（Duck）

```java
public class RubberDuck extends Duck {
	@Override
	public void quack() {
		//覆盖成吱吱叫
	}

	@Override
	public void display() {
	}

}
```

这样橡皮鸭也实现了，可是问题出现了，橡皮鸭是不会飞的，但是我们的橡皮鸭却飞起来了，怎么办？这也简单，把橡皮鸭的fly()方法覆盖掉就好了，变成什么事情都不做。这样橡皮鸭就不会再飞了。

这时老板过来看到我们的游戏很不错，但是觉得游戏中鸭子的种类太少了，还可以再加一个诱饵鸭（DecoyDuck），诱饵鸭是假鸭不会飞也不会叫，我们只需要覆盖诱饵鸭的quack()和fly()方法，变成什么都不做。

我们的噩梦开始了，老板要求每周都增加一批新的鸭子子类，至于具体怎么加还没想好。每当有新的鸭子子类出现，我们就得检查并可能需要覆盖fly()和quark()方法。看来继承不适合解决我们的问题，我们需要一个更清晰的方法，不至少陷入无尽的痛苦中。

### 使用接口

接下来我们考虑能否利用接口解决我们的问题，我们可以把fly()取出来，放进一个Flyable接口中。这样一来，只有会飞的鸭子才实现此接口。同样的方法，也可以用来设计一个Quackable接口，因为不是所有的鸭子都会叫。

现在我们知道使用继承有一些缺失，因为改变鸭子的行为会影响所有种类的鸭子，而这并不恰当。Flyable与Quackable接口一开始似乎还挺不错，解决了问题，但是Java的接口不具有实现代码，所以实现接口无法达到代码的复用。这意味着：无论何时你需要修改某个行为，你必须得往下追踪并修改每一个定义此行为的类，一不小心，可能造成新的错误。

幸运的是，有一个设计原则，正适用于此状况。

### 设计原则

找出应用中可能需要变化之处，把它们独立出来，不要和那些不需要变化的代码混在一起。

结果如何？代码变化之后，现其不意的部分变得很少，系统变得更有弹性。下面是按照**设计原则**实现的代码清单:

```java
public interface FlyBehavior {
	public void fly();
}
```

这里实现了所有有翅膀的鸭子飞行动作

```java
public class FlyWithWings implements FlyBehavior {

	@Override
	public void fly() {
		//实现鸭子的飞行动作
		System.out.println("I'm flying!!");
	}

}
```

这里实现了所有不会飞的鸭子的动作

```java
public class FlyNoWay implements FlyBehavior {

	@Override
	public void fly() {
		//什么都不做，不会飞
		System.out.println("I'm can't fly!!");
	}

}
```

这里我们只实现FlyBehavior接口，同样的方法可以实现QuackBehavior接口。

所有飞行类都实现 FlyBehavior 接口，所有新的飞行类都必须实现fly()方法。

这样的设计，可以让飞行的动作被其他的对象复用，因为这些行为已经与鸭子类无关了。而我们可以新增一些行，不会影响到既有的行为类，也不会影响有**使用**到飞行行为的鸭子类。

### 整合鸭子的行为

```java
public abstract class Duck {
	private FlyBehavior flyBehavior;	//行为变量被声明为接口类型
	
	public void performFly(){	//取代fly()方法
		flyBehavior.fly();
	}	
	
	public void swim(){}
	
	public abstract void display();
	
	//鸭子的其他方法...
}
```

```java
public class MallardDuck extends Duck {

	public MallardDuck() {
		flyBehavior = new FlyWithWings();
	}

	@Override
	public void display() {
		System.out.println("I'm a real Mallard duck");
	}

}
```

### 测试Duck类

```java
public class MiniDuckSimulator {

	public static void main(String[] args) {
		Duck mallard = new MallardDuck();
		mallard.performFly();
	}
}
```

运行结果：`I'm flying!!`

### 动态设定行为

我们还可以实现动态设定鸭子的行为，而不是在鸭子的构造器内实例化，只需要在Duck类中加入一个新的方法，程序清单如下：

```java
public void setFlyBehavior(FlyBehavior fb) {
		flyBehavior = fb;
	}
```

此后，我们可以随时调用此方法改变鸭子的行为。

我们重新设计的SimUDuck程序使用的模式就是**策略模式**，我们来看一下**策略模式**的定义。

## 策略模式定义

定义了算法族，分别封装起来，让它们之间可以互相替换，此模式让算法的变化独立于使用算法的客户。

到此，我们介绍了设计模式的使用与优点，并通过一个例子了解模式如何运作。使用模式最好的方式是：把模式装进脑子中，然后在你的设计和已有的应用中，寻找何处可以使用这些模式。