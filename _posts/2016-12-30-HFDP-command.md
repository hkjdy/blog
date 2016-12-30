---
date: 2016/12/30 星期五  15:18:46+0800
layout: post
title: HeadFirst设计模式-命令模式
thread: 164
categories: 读书笔记
tags:  设计模式 命令模式
---

本文通过遥控器系统的设计来介绍 **命令模式**。命令模式就是把方法调用封装起来。

## 遥控器

我们设计的是一款家电自动化遥控器的API，能够控制电灯、风扇、热水器、音响设备和其他类似的可控制装置。

![telecontroller](/assets/telecontroller/telecontroller1.png)

从图中可以看出我们要设计的遥控器能用一组按钮（ON&OFF）来控制一台家电设备，例如：第一组按钮可以控制电灯的开关。

## 已知条件

我们有家电设备厂商提供的厂商类似类下面这些(这里只列举其中的两种)：

### Light厂商类

```java
public class Light {
	
	private String name;
	
	public Light(String name) {
		this.name = name;
	}

	public void on() {
		System.out.println(name + " Light is on");
	}
	
	public void off() {
		System.out.println(name + "Light is off");
	}
}
```

### OutdoorLight厂商类

```java
public class OutdoorLight {
	public void on() {}
	
	public void off() {}
}

```

除了on()跟off()方法以外，其他类还可能还会有一些方法像是dim()、setTemperature()、setVolumn()、setDirection()。而且似乎将来还会有更多的厂商类，而且每个类还会有各式各样的方法。所以我们不想让遥控器包含一大堆if语句，例如：

```java
if (slot == Light) {
} else if (slot == TV) {
} else if (slot == Fan) {
}
```

因为，只要有新厂商类进来，就必须修改代码，这会造成潜在的错误，而且工作没完没了。所以，我们考虑使用 **命令模式** 来改善我们的设计。

在设计中采用 **命令对象**。利用命令对象，把请求（例如打开电灯）封装成一个特定对象（例如客厅电灯对象）。所以，如果每个按钮都存储一个命令对象，那么当按钮被按下的时候，就可以请命令对象做相关的工作。遥控器不需要知道工作内容是什么，只要有个命令对象能和正确的对象沟通，把事情做好就可以了。

下面来看具体的实现：

### 实现命令接口

```java
public interface Command {
	public void execute();
}
```

### 实现一个打开电灯的命令

```java
public class LightOnCommand implements Command {

	Light light;
	
	public LightOnCommand(Light light) {
		this.light = light;
	}

	//这个方法调用接收对象（我们正在控制的电灯）的on()方法
	@Override
	public void execute() {
		light.on();
	}

}

```

### 实现一个关闭电灯的命令

```java
public class LightOffCommand implements Command {

	Light light;
	
	public LightOffCommand(Light light) {
		this.light = light;
	}

	@Override
	public void execute() {
		light.off();
	}

}
```

### 合作命令对象

```java
public class SimpleRemoteControl {
	Command slot;
	
	public SimpleRemoteControl() {}
	
	public void setComand(Command command) {
		slot = command;
	}
	
	public void buttonWasPressed() {
		slot.execute();
	}
}
```

### 遥控器使用的简单测试

```java
public class RomoteControlTest {

	public static void main(String[] args) {
		SimpleRemoteControl remote = new SimpleRemoteControl();
		
		Light light = new Light("living room");
		
		LightOnCommand lightOn = new LightOnCommand(light);
		
		remote.setComand(lightOn);
		
		remote.buttonWasPressed();
	}

}
```

### 运行结果

```
living room Light is on
```

## 定义命令模式

**将“请求”封装成对象，以便使用不同的请求、队列或者日志来参数化其他对象。命令模式也支持可撤销的操作。**

## 定义命令模式类图

![telecontroller](/assets/telecontroller/telecontroller2.png)

## 将命令指定到按钮

我们打算将遥控器的每一组按钮，对应到一个命令这样就让遥控器变成“调用者”。当按下按钮，相应命令对象的execute()方法就会被调用，其结果就是，接收者（例如：电灯、电扇、音响）的动作被调用。

## 实现遥控器

```java
public class NoCommand implements Command {

	@Override
	public void execute() {
	}
}
```

```java
public class RemoteControl {
	Command[] onCommands;
	Command[] offCommands;
	
	public RemoteControl() {
		onCommands = new Command[7];
		offCommands = new Command[7];
		
		Command noCommand = new NoCommand();
		
		for(int i=0; i<7;i++) {
			onCommands[i] = noCommand;
			offCommands[i] = noCommand;
		}
		
	}

	public void setCommand(int slot, Command onCommand, Command offCommand) {
		onCommands[slot] = onCommand;
		offCommands[slot] = offCommand;
	}
	//当按下开按钮，会调用此方法
	public void onButtonWasPushed(int slot) {
		onCommands[slot].execute();
	}
	//当按下关按钮，会调用此方法
	public void offButtonWasPushed(int slot) {
		offCommands[slot].execute();
	}
}

```

## 测试遥控器

遥控器的工作差不多已经完成，我们剩下的事情就是运行测试。

```java
public class RemoteLoader {

	public static void main(String[] args) {
		RemoteControl remoteControl = new RemoteControl();
		
		Light livingRoomLight = new Light("Living Room");
		Light kitchenLight = new Light("Kitchen");
		
		LightOnCommand livingRoomLightOn = new LightOnCommand(livingRoomLight);
		LightOffCommand livingRoomLightOff = new LightOffCommand(livingRoomLight);
		
		LightOnCommand kitchenLightOn = new LightOnCommand(kitchenLight);
		LightOffCommand kitchenLightOff = new LightOffCommand(kitchenLight);
		
		remoteControl.setCommand(0, livingRoomLightOn, livingRoomLightOff);
		remoteControl.setCommand(1, kitchenLightOn, kitchenLightOff);
		
		remoteControl.onButtonWasPushed(0);
		remoteControl.offButtonWasPushed(0);
		
		remoteControl.onButtonWasPushed(1);
		remoteControl.offButtonWasPushed(1);
		
	}
}

```

运行结果

```
Living Room Light is on
Living RoomLight is off
Kitchen Light is on
KitchenLight is off
```

## 总结

下面的类图提供了设计的全貌:

![telecontroller](/assets/telecontroller/telecontroller3.png)