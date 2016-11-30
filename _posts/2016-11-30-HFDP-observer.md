---
date: 2016/11/30 星期三  16:21:21+0800
layout: post
title: HeadFirst设计模式-观察者模式
thread: 164
categories: 读书笔记
tags:  设计模式 观察者模式
---

本章首先通过一个气象站系统的实现来学习观察者模式的原理，最后简单介绍Java内置的观察者模式。

## 定义

**定义了对象之间的一对多依赖，这样一来，当一个对象改变状态时，它的所有依赖者都会收到通知并自动更新。**

## 类比

报纸和杂志订阅，出版者(主题)+订阅者(观察者)=观察者模式

1. 报社的业务就是出版报纸。

2. 向某家报社订阅报纸，只要他们有新报纸出版，就会给你送来。只要你是他们的订户，你就会一直收到新报纸。

3. 当你不想再看报纸的时候，取消订阅，他们就不会再送新报纸来。

4. 只要报社还在运营，就会一直有人(或单位)向他们订阅或取消订阅报纸。

这里的报社就相当于是观察者模式中的主题（Subject）,而订阅用户就相当于是观察者（Observer）。

## 气象站应用

此系统包含三个部分，分别是气象站（获取实际气象数据的物理装置）、WeatherData对象（追踪来自气象站的数据，并更新布告板）和布告板（显示目前天气状况给用户看）。

![station](/assets/observer/observer1.png)

WeatherData对象知道如何跟物理气象站联系，以取得更新的数据。所以我们不需要关心气象站这一部分，我们的工作就是利用WeatherData对象取得数据，并更新布告板。

### 设计气象站

![station](/assets/observer/observer2.png)

### 实现气象站

#### 主题接口

```java
public interface Subject {
	//注册观察者
	public void registerObserver(Observer o);
	//删除观察者
	public void removeObserver(Observer o);
	//当主题状态改变时，这个方法会被调用，以通知所有的观察者
	public void notifyObservers();
}

```

#### 观察者接口

```java
public interface Observer {
	public void update(float temp, float humidity, float pressure);
}
```

#### DisplayElement接口

```
public interface DisplayElement {
	//当布告板需要显示时，调用此方法。
	public void display();
}
```

#### 实现主题接口

```java
public class WeatherData implements Subject {

	//加上一个List来记录观察者,此List是在构造器中建立的
	private List<Observer> observers;
	private int temperature;
	private float humidity;
	private float pressure;
	
	
	public WeatherData() {
		observers = new ArrayList<Observer>();
	}

	@Override
	public void registerObserver(Observer o) {
		observers.add(o);
	}

	@Override
	public void removeObserver(Observer o) {
		int i = observers.indexOf(o);
		if(i>=0) {
			observers.remove(i);
		}
	}
	
	public void setMeasurements(int temperature, float humidity, float pressure) {
		this.temperature = temperature;
		this.humidity = humidity;
		this.pressure = pressure;
		measurementsChanged();
	}

	//当从气象站得到更新观测值时，我们通知观察者
	public void measurementsChanged() {
		notifyObservers();
	}
	
	@Override
	public void notifyObservers() {
		for(Observer observer : observers) {
			observer.update(temperature, humidity, pressure);
		}
	}
}

```

#### 实现布告板

```java
public class CurrentConditionsDisplay implements Observer, DisplayElement {

	private int temperature;
	private float humidity;
	private Subject weatherData;
	
	
	public CurrentConditionsDisplay(Subject weatherData) {
		this.weatherData = weatherData;
		weatherData.registerObserver(this);
	}

	@Override
	public void display() {
		System.out.println("当前温度:" + temperature
				+ "℃ 湿度:" + humidity+ "%");
	}

	@Override
	public void update(int temperature, float humidity, float pressure) {
		this.temperature = temperature;
		this.humidity = humidity;
		display();
	}

}

```

ForecastDisplay 布告板可以参照CurrentConditionsDisplay布告板来实现。程序清单如下：

```java
public class ForecastDisplay implements Observer, DisplayElement{

	private float pressure;
	private Subject weatherData;
	
	
	public ForecastDisplay(Subject weatherData) {
		this.weatherData = weatherData;
		weatherData.registerObserver(this);
	}

	@Override
	public void display() {
		System.out.println("当前气压:" + pressure);
	}

	@Override
	public void update(int temperature, float humidity, float pressure) {
		this.pressure = pressure;
		display();
	}
}
```

#### 启动气象站

```java
public class WeatherStation {
	public static void main(String[] args) {
		WeatherData weatherData = new WeatherData();
		
		CurrentConditionsDisplay currentDisplay = new CurrentConditionsDisplay(weatherData);
		
		weatherData.setMeasurements(35, 65, 30.4f);
		
		weatherData.setMeasurements(20, 60, 35.5f);
		
	}
}

```

#### 运行结果：

```
当前温度:35℃ 湿度:65.0%
当前气压:30.4
当前温度:20℃ 湿度:60.0%
当前气压:35.5
```

我们看到，一旦weatherData数据发生变化，两个布告板都会接收到最新的数据，并且显示出来。

## 使用Java内置的观察者模式

到目前为止，我们已经从无到有地完成了观察者模式，但是，JAVA API有内置的观察者模式。java.util包内包含最基本的Observer接口与Observable类，这和我们的Subject接口与Observer接口很相似。Observer接口与Observerable类使用上更方便，因为许多功能都已经事先准备好了。你甚至可以使用推(push)或拉(pull)的方式传送数据。

接下来我们来看看修改后的气象站设计图

![station](/assets/observer/observer3.png)

### Java内置的观察者模式如何运作

#### 如何把对象变成观察者

如同以前一样，实现观察者接口（Observer），然后调用任何Observable对象的addObserver()方法。不想再当观察者时，调用deleteObserver()方法就可以了。

#### 可观察者要如何送出通知

首先，你需要复用扩展java.util.Observable接口产生“可观察者”类，然后，需要两个步骤：

1. 先调用setChanged()方法，标记状态已经改变的事实。

2. 然后调用notifyObservers()方法中的一个：

notifyObservers() 或 notifyObservers(Object arg)

#### 观察者如何接收通知

同以前一样，观察者实现了更新的方法，但是方法的签名不太一样：

```java
update(Observable o, Object arg)
```

### 利用内置的支持重做气象站

#### 实现主题:WeatherData

```
public class WeatherData extends Observable {
	private int temperature;
	private float humidity;
	private float pressure;
	
	//构造器不再需要为记住观察而建立数据结构了
	public WeatherData() {}
	
	public void measurementsChanged() {
		this.setChanged();
		this.notifyObservers();
	}
	
	public void setMeasurements(int temperature, float humidity, float pressure) {
		this.temperature = temperature;
		this.humidity = humidity;
		this.pressure = pressure;
		measurementsChanged();
	}

	public int getTemperature() {
		return temperature;
	}

	public float getHumidity() {
		return humidity;
	}

	public float getPressure() {
		return pressure;
	}
	
}
```

#### 实现CurrentConditionsDisplay布告板

```
public class CurrentConditionsDisplay implements DisplayElement, Observer {

	Observable observable;
	private int temperature;
	private float humidity;
	
	
	
	public CurrentConditionsDisplay(Observable observable) {
		this.observable = observable;
		observable.addObserver(this);
	}

	@Override
	public void update(Observable obs, Object arg) {
		if(obs instanceof WeatherData) {
			WeatherData weatherData = (WeatherData)obs;
			this.temperature = weatherData.getTemperature();
			this.humidity = weatherData.getHumidity();
			display();
		}
	}

	@Override
	public void display() {
		System.out.println("当前温度:" + temperature
				+ "℃ 湿度:" + humidity+ "%");
	}

}
```

同样原理实现 ForecastDisplay 布告板

```
public class ForecastDisplay implements Observer, DisplayElement {

	private float currentPressure = 29.92f;
	private float lastPressure;
	
	private WeatherData weatherData;
	
	
	public ForecastDisplay(Observable observable) {
		weatherData = (WeatherData)observable;
		observable.addObserver(this);
	}
	
	@Override
	public void display() {
		System.out.println("当前气压：" + currentPressure);
	}

	@Override
	public void update(Observable observable, Object arg) {
		if (observable instanceof WeatherData) {
			lastPressure = this.currentPressure;
			this.currentPressure = weatherData.getPressure();
			this.display();
		}
	}

}
```

#### 启动气象站

````
public class WeatherStation {

	public static void main(String[] args) {
		WeatherData weatherData = new WeatherData();
		
		CurrentConditionsDisplay currentDisplay = new CurrentConditionsDisplay(weatherData);
		
		ForecastDisplay forecastDisplay = new ForecastDisplay(weatherData);
		
		weatherData.setMeasurements(35, 65, 30.4f);
		
		weatherData.setMeasurements(20, 60, 35.5f);
	}

}

````

#### 运行结果

```
当前气压：30.4
当前温度:35℃ 湿度:65.0%
当前气压：35.5
当前温度:20℃ 湿度:60.0%
```

得到了我们想要的结果。本章通过气象站系统的设计来了解观察者模式的原理，这里只实现了其中两个布告板，我们可以根据需要添加更多的布告板，原理与之前的实现相同。
