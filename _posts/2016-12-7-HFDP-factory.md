---
date: 2016/12/7 星期三  21:04:51+0800
layout: post
title: HeadFirst设计模式-工厂方法模式
thread: 164
categories: 读书笔记
tags:  设计模式 工厂方法模式
---

除了使用new操作符之外，还有更多创建对象的方法，本章就来介绍如何使用工厂模式来创建新的对象。

当使用new创建对象时，你是在实例化一个具体类，所以用的是实现，而不是接口。这违背了 **针对接口编程，而不是针对实现编程** 的原则。代码绑着具体类会导致代码更脆弱，更缺乏弹性。

```java
Duck duck = 			//使用接口让代码具有弹性
	new MallardDuck();	//但是还得建立具体类的实例
```

当有一群具体类时，通常会写出这样的代码：

```java
Duck duck;

if (picnic) {
	duck = new MallardDuck();
} else if (hunting) {
	duck = new DecoyDuck();
} else if (inBathTub) {
	duck = new RubberDuck();
}
```

这里有一些要实例化的具体类，究竟实例化哪个类，要在运行时由一些条件来决定。

当看到这样的代码，一旦有变化或扩展，就必须重新打开这段代码进行检查和修改。通常这样修改过的代码将造成部分系统更难维护和更新，而且也更容易犯错。

下面我们通过披萨店系统的实现来介绍如果解决上面的问题。

## 披萨店

我们要实现一个从披萨店订购披萨的功能，首先定义Pizza类。

```java
public class Pizza {
	
	public void prepare() {}
	
	public void bake() {}
	
	public void cut() {}
	
	public void box() {}
}
```

接下来定义PizzaStore类，我们通过PizzaStore类来订购披萨。

```java
public class PizzaStore {
	
	//订购披萨方法
	public Pizza orderPizza() {
		Pizza pizza = new Pizza();
		
		pizza.prepare();	//准备一个披萨
		pizza.bake();		//烘烤
		pizza.cut();		//切片
		pizza.box();		//包装
		
		return pizza;
	}
}
```

这里的披萨店只能提供一种披萨，我们希望有更多的披萨可供订购，于是我们给orderPizza()添加一个参数代表披萨类型:

```java
public class PizzaStore {
	
	public Pizza orderPizza(String type) {
		Pizza pizza = new Pizza();		//默认类型
		
		if (type.equals("cheese")) {
			pizza = new CheesePizza();
		} else if (type.equals("greek")) {
			pizza = new GreekPizza();
		} else if (type.equals("pepperoni")) {
			pizza = new PepperoniPizza();
		}
		
		pizza.prepare();
		pizza.bake();
		pizza.cut();
		pizza.box();
		
		return pizza;
	}

}
```

这样我们就可以有更多的披萨可供订购。其中CheesePizza、GreekPizza和PepperoniPizza是具体的披萨类型。

根据一段时间披萨的销量统计，我们需要调整披萨的菜单，去掉销量不好的披萨，同时增加一些其他种类的披萨。程序清单如下：

```java
public Pizza orderPizza(String type) {
		Pizza pizza = new Pizza();
		
		if (type.equals("cheese")) {
			pizza = new CheesePizza();
		} 
//		销量不好的披萨要从菜单中去掉
//		else if (type.equals("greek")) {
//			pizza = new GreekPizza();
//		} else if (type.equals("pepperoni")) {
//			pizza = new PepperoniPizza();
//		} 
//		新添加的披萨
		else if (type.equals("clam")) {
			pizza = new ClamPizza();
		} else if (type.equals("veggie")) {
			pizza = new VeggiePizza();
		}
		
		//这里不需要改变
		pizza.prepare();
		pizza.bake();
		pizza.cut();
		pizza.box();
		
		return pizza;
	}
```

很明显，如果实例化“某些”具体类，将使orderPizza()出问题，我们要不停的修改这个方法。所以我们想到要把会发生变化的代码(创建不同种类的披萨)封装起来。我们要把下面这部分创建Pizza对象的代码提取出来。

```java
if (type.equals("cheese")) {
	pizza = new CheesePizza();
} else if (type.equals("clam")) {
	pizza = new ClamPizza();
} else if (type.equals("veggie")) {
	pizza = new VeggiePizza();
}
```

修改后的代码清单如下：

```java
//把创建Pizza对象的工作放到这里
public class SimplePizzaFactory {
	public static Pizza createPizza(String type) {
		Pizza pizza = new Pizza();
		if (type.equals("cheese")) {
			pizza = new CheesePizza();
		} else if (type.equals("clam")) {
			pizza = new ClamPizza();
		} else if (type.equals("veggie")) {
			pizza = new VeggiePizza();
		}
		return pizza;
	}
}

```

```java
public class PizzaStore {
	private SimplePizzaFactory factory;

	public PizzaStore(SimplePizzaFactory factory) {
		this.factory = factory;
	}

	public Pizza orderPizza(String type) {
		//通过简单传入订单类型来使用工厂创建披萨
		Pizza pizza = factory.createPizza(type);
		
		pizza.prepare();
		pizza.bake();
		pizza.cut();
		pizza.box();
		
		return pizza;
	}

}

```

我们用简单工厂来替换new操作符来创建披萨对象，这里不再使用具体实例化。

我们来看看新的披萨店类图:

![pizza](/assets/pizza/pizza1.png)

## 加盟披萨店

我们的披萨很受欢迎，所以我们打算再开几家加盟店来满足不同地区的顾客对披萨的需求。而且我们还要根据不同地区顾客口味的差异提供不同风味的披萨。

如果利用SimplePizzaFactory，写出多种不同的工厂，分别是NYPizzaFactory、ChicagoPizzaFactory、CaliforniaPizzaFactory，那么各地加盟店都有适合的工厂可以使用，这是一种做法。实现代码如下:

```java
//纽约的披萨工厂生产的披萨
NYPizzaFactory nyFactory = new NYPizzaFactory();
PizzaStore nyStore = new PizzaStore(nyFactory);
nyStore.orderPizza("Veggie");

//芝加哥的披萨工厂生产的披萨
ChicagoPizzaFactory chicagoFactory = new ChicagoPizzaFactory();
PizzaStore chicagoStore = new PizzaStore(chicagoFactory);
chicagoStore.orderPizza("Veggie");
```

现在加盟店可以使用我们提供的工厂来生产披萨。但是处理订单的其他部分，却开始采用他们自创的流程：烘烤的做法有些差异、不要切片、使用其他厂商的盒子。我们不希望以上事情的发生，我们希望所有加盟店处理订单的方式能够保持一致，同时保证不同的地区能够生产不同风味的披萨。

我们的做法就是把createPizza()方法放回到PizzaStore中，不过把它设置成**抽象方法**，然后为每个地区风味创建一个PizzaStore的子类。

```java
public abstract class PizzaStore {

	public final Pizza orderPizza(String type) {
		//createPizza()方法从工厂对象中移回PizzaStore
		Pizza pizza = createPizza(type);
		
		pizza.prepare();
		pizza.bake();
		pizza.cut();
		pizza.box();
		
		return pizza;
	}
	
	//现在把工厂对象移到这个方法中
	public abstract Pizza createPizza(String type);
}
```

**纽约的加盟店**

```java
public class NYStylePizzaStore extends PizzaStore {

	@Override
	public Pizza createPizza(String type) {
		
		if (type.equals("cheese")) {
			return new NYStyleCheesePizza();
		} else if (type.equals("clam")) {
			return new NYStyleClamPizza();
		} else if (type.equals("veggie")) {
			return new NYStyleVeggiePizza();
		}
		return null;
	}
}
```

**芝加哥的加盟店**

```java
public class ChicagoStylePizzaStore extends PizzaStore {

	@Override
	public Pizza createPizza(String type) {
		if (type.equals("cheese")) {
			return new ChicagoStyleCheesePizza();
		} else if (type.equals("clam")) {
			return new ChicagoStyleClamPizza();
		} else if (type.equals("veggie")) {
			return new ChicagoStyleVeggiePizza();
		}
		return null;
	}
}
```

现在有一个PizzaStore作为超类，让每个地区类型（NYPizzaStore、ChicagoPizzaStore）都继承这个PizzaStore，每个子类各自决定如何制造披萨，只需要实现 createPizza()方法。同时我们把orderPizza()方法声明成final的，以防止被子类覆盖。

## 订购披萨

下面我通过订购一个披萨来看看如何根据订单生产披萨

```java
public class Client {
	public static void main(String[] args) {
		PizzaStore nyStylePizzaStore = new NYStylePizzaStore();
		nyStylePizzaStore.orderPizza("cheese");
	}
}
```

1.首先需要一个纽约披萨店：

```java
PizzaStore nyStylePizzaStore = new NYStylePizzaStore();
```

2.现在有了一个披萨店，可以下订单了：

```java
nyStylePizzaStore.orderPizza("cheese");
```

3.orderPizza()方法于是调用createPizza()方法:

```java
Pizza pizza = createPizza("cheese");
```

工厂方法createPizza()是在子类中实现的。在这个例子中，它会返回纽约芝士披萨。

4.最后，披萨必须经过下列的处理才算完成orderPizza():

```java
pizza.prepare();
pizza.bake();
pizza.cut();
pizza.box();
```

到此为止我们统一了披萨订单的处理过程，同时又保证不同地区的披萨店可以生产不同风味的披萨。接下来我们要实现具体的披萨子类。

首先，我们要完善Pizza超类。程序清单如下：

```java
public abstract class Pizza {
	
	String name;	//披萨名称
	String dough;	//面团类型
	String sauce;	//酱料类型
	List<String> toppings = new ArrayList<String>();	//一套佐料
		
	public void prepare() {
		System.out.println("Preparing " + name);
		System.out.println("Tossing dough...");
		System.out.println("Adding sauce...");
		for(String topping : toppings) {
			System.out.println(topping);
		}
	}
	
	public void bake() {
		System.out.println("Bake for 25 minutes at 350");
	}
	
	public void cut() {
		System.out.println("Cutting the pizza into diagonal slices");
	}
	
	public void box() {
		System.out.println("Place pizza in official PizzaStore box");
	}
	
	public String getName() {
		return name;
	}
}
```

接下来实现一些子类

```java
public class NYStyleCheesePizza extends Pizza {

	public NYStyleCheesePizza() {
		name = "NY Style Sauce and Cheese Pizza";
		dough = "Thin Crust Douth";
		sauce = "Marinara Sauce";
		toppings.add("Grated Reggiano Cheese");
	}
}
```

```java
public class ChicagoStyleCheesePizza extends Pizza {
	public ChicagoStyleCheesePizza() {
		name = "Chicago Style Deep Dish Cheese Pizza";
		dough = "Extra Thick Crust Dough";
		sauce = "Plum Tomato Sauce";
		toppings.add("Shredded the pizza into square slices");
	}
}

```

最后我们来订购两个披萨(纽约披萨和芝加哥披萨)

```java
public class Client {
	public static void main(String[] args) {
		PizzaStore nyStore = new NYStylePizzaStore();
		PizzaStore chicagoStore = new ChicagoStylePizzaStore();
		
		Pizza pizza = nyStore.orderPizza("cheese");
		System.out.println("ordered a " + pizza.getName() );
		
		pizza = chicagoStore.orderPizza("cheese");
		System.out.println("and ordered a " + pizza.getName());
		
	}
}
```

运行结果

```
Preparing NY Style Sauce and Cheese Pizza
Tossing dough...
Adding sauce...
Grated Reggiano Cheese
Bake for 25 minutes at 350
Cutting the pizza into diagonal slices
Place pizza in official PizzaStore box
ordered a NY Style Sauce and Cheese Pizza
Preparing Chicago Style Deep Dish Cheese Pizza
Tossing dough...
Adding sauce...
Shredded the pizza into square slices
Bake for 25 minutes at 350
Cutting the pizza into diagonal slices
Place pizza in official PizzaStore box
and ordered a Chicago Style Deep Dish Cheese Pizza
```

## 认识工厂方法模式

所有工厂模式都用来封装对象的创建，工厂方法模式通过让子类决定该创建的对象是什么，来达到将对象创建的过程封装的目的。

## 工厂方法模式定义

**定义了一个创建对象的接口，但由子类决定要实例化的类是哪一个。工厂方法让类把实例化推迟到子类。**

工厂方法模式能够封装具体类型的实例化。看看下面的类图，抽象的Creator(相当于PizzaStore)提供了一个创建对象的方法的接口（factoryMethod），也称为 **工厂方法** 。在抽象的Creator中，任何其他实现的方法，都可能使用到这个工厂方法所制造出来的产品，但只有子类真正实现这个工厂方法并创建产品。

![pizza](/assets/pizza/pizza2.png)

## 回到披萨店

披萨店的设计变得很棒：具有弹性的框架，而且遵循设计原则。

随着加盟店的增多，我们要保证每家加盟店都能使用新鲜、高质量的原料，避免一些加盟店使用低价原料来增加利润。我们打算建造一家生产原料的工厂，并将原料运送到各家加盟店。同时我们还要根据加盟店区域差异准备不同的原料。因为纽约的红酱料跟芝加哥的红酱料是不一样的。

![pizza](/assets/pizza/pizza3.png)

我们有相同的产品家族，但是制作方式根据区域的不同而有差异。

现在，我们要建造一个工厂来生产原料；这个工厂将负责创建原料家族中的每一种原料。也就是说，工厂将需要生产面团、酱料、芝士等。接下来是我们的实现：

- 开始先为工厂定义一个接口，这个接口负责创建所有的原料：

```java
public interface PizzaIngredientFactory {
	public Dough createDough();
	public Sauce createSauce();
	public Cheese createCheese();
	public Veggies[] createVeggies();
	public Pepperoni createPepperoni();
	public Clams createClam();
}
```

- 创建纽约原料工厂,同样方法还可以创建芝加哥原料工厂

```java
public class NYPizzaIngredientFactory implements PizzaIngredientFactory {

	@Override
	public Dough createDough() {
		return new ThinCrustDough();
	}

	@Override
	public Sauce createSauce() {
		return new MarinaraSauce();
	}

	@Override
	public Cheese createCheese() {
		return new ReggianoCheese();
	}

	@Override
	public Veggies[] createVeggies() {
		Veggies veggies[] = {new Garlic(), new Onion(), new Mushroom(), new RedPepper()};
		return veggies;
	}

	@Override
	public Pepperoni createPepperoni() {
		return new SlicedPepperoni();
	}

	@Override
	public Clams createClam() {
		return new FreshClams();
	}

}
```

原料工厂准备好我们就可以生产披萨了。

## 重做披萨

```java
public abstract class Pizza {
	
	String name;	//披萨名称
	Dough dough;	//面团类型
	Sauce sauce;	//酱料类型
	Veggies veggies[];
	Cheese cheese;
	Pepperoni pepperoni;
	Clams clam;
	
	public abstract void prepare();
	
	public void bake() {
		System.out.println("Bake for 25 minutes at 350");
	}
	
	public void cut() {
		System.out.println("Cutting the pizza into diagonal slices");
	}
	
	public void box() {
		System.out.println("Place pizza in official PizzaStore box");
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}
	
}
```

现在有了一个抽象披萨，可以开始制作纽约和芝加哥风味的披萨了。从今以后，加盟店必须直接从工厂取得原料。

我们前面写过工厂方法的代码，有NYCheesePizza和ChicagoCheesePizza类。比较一下这两个类，唯一的差别在于使用区域性的原料，至于披萨的做法都一样，其他的披萨也是如此。

所以，其实我们不需要设计两个不同的类来处理不同风味的披萨，让原料工厂处理这种区域差异就可以了。下面是CheesePizza和ClamPizza：

```java
public class CheesePizza extends Pizza {

	PizzaIngredientFactory factory;
	
	public CheesePizza(PizzaIngredientFactory factory) {
		this.factory = factory;
	}

	@Override
	public void prepare() {
		System.out.println("Preparing " + name);
		dough = factory.createDough();
		sauce = factory.createSauce();
		cheese = factory.createCheese();
	}
	
}
```

```java
public class ClamPizza extends Pizza {

	PizzaIngredientFactory factory;
	
	public ClamPizza(PizzaIngredientFactory factory) {
		this.factory = factory;
	}

	@Override
	public void prepare() {
		System.out.println("Preparing " + name);
		dough = factory.createDough();
		sauce = factory.createSauce();
		cheese = factory.createCheese();
	}

}
```

Pizza的代码利用相关的工厂生产原料。所生产的原料依赖所使用的工厂，Pizza类根本不关心这些原料，它只知道如何制作披萨。现在，Pizza和区域原料之间被解耦，无论原料工厂在哪里，Pizza类都可以国易地复用，完全没有问题。

重新实现纽约披萨店

```java
public class NYStylePizzaStore extends PizzaStore {

	@Override
	public Pizza createPizza(String type) {
		
		Pizza pizza = null;
		
		PizzaIngredientFactory factory = new NYPizzaIngredientFactory();
				
		if (type.equals("cheese")) {
			pizza = new CheesePizza(factory);
			pizza.setName("New York Style Cheese Pizza");
		} else if (type.equals("clam")) {
			pizza = new ClamPizza(factory);
			pizza.setName("New York Style Clam Pizza");
		} 
		return pizza;
	}
}

```

订购纽约披萨店的披萨

```java
public class Client {
	public static void main(String[] args) {
		PizzaStore nyStore = new NYStylePizzaStore();
		Pizza pizza = nyStore.orderPizza("cheese");
	}
}
```

运行结果

```java
Preparing New York Style Cheese Pizza
Bake for 25 minutes at 350
Cutting the pizza into diagonal slices
Place pizza in official PizzaStore box

```

运行结果分析

1.首先我们需要一个纽约披萨店：

```java
PizzaStore nyStore = new NYStylePizzaStore();
```

2.现在有一个披萨店了，可以接受订单：

```java
Pizza pizza = nyStore.orderPizza("cheese");
```

3.orderPizza()方法首先调用createPizza()方法

```java
Pizza pizza = createPizza("cheese");
```

4.当createPizza()方法被调用时，也就开始涉及原料工厂了：

```java
Pizza pizza = new CheesePizza(factory);
```

5.接下来需要准备披萨。一旦调用了prepare()方法，工厂将被要求准备原料：

```java
public void prepare() {
		System.out.println("Preparing " + name);
		dough = factory.createDough();
		sauce = factory.createSauce();
		cheese = factory.createCheese();
	}
```
6.最后，我们得到了准备好的披萨，orderPizza()就会接着烘烤、切片、装盒。


## 定义抽象工厂模式

**提供一个接口，用于创建相关或相依赖对象的家族，而不需要明确指定具体类。**

抽象工厂允许客户使用抽象的接口来创建一组相关的产品，而不需要关心实际产出的具体产品是什么。这样一来，客户就从具体的产品中被解耦。

## 总结

- 所有的工厂都是用来封装对象的创建。

- 简单工厂，虽然不是真正的设计模式，但仍不失为一个简单的方法，可以将客户程序从具体类解耦。

- 工厂方法使用继承：把对象的创建委托给子类，子类实现工厂方法来创建对象。

- 抽象工厂使用对象组合：对象的创建被实现在工厂接口所暴露出来的方法中。

- 所有工厂模式都通过减少应用程序和具体类之间的依赖促进松耦合。

- 工厂方法允许类将实例化延迟到子类进行。

- 抽象工厂创建相关的对象家族，而不需要依赖它们的具体类。

- 工厂是很有威力的技巧，帮助我们针对抽象编程，而不要针对具体类编程。
