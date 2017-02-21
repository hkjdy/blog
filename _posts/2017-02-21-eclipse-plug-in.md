---
date: 2017/2/21 星期二  12:02:22+0800
layout: post
title: 简单Eclipse插件开发-统计Eclipse启动消耗时间
thread: 164
categories: eclipse
tags:  eclipse插件
---

# 一、Eclipse插件开发

## 1.下载并安装JDK和Eclipse

这里使用的是JDK 1.8、Eclipse版本为：Mars.2 Release (4.5.2)。

## 2.新建项目

### 1) 打开Eclipse，依次选择 `File -> New -> Project -> Plug-in Project` 新建一个Plug-in项目，如下：

![JVM](/assets/eclipse-plug-in/01.png)

这里模版选择`Hello, World Command` 如下：

![JVM](/assets/eclipse-plug-in/02.png)

### 2) 新建一个ShowTime类，输入如下代码：

```java
public class ShowTime implements IStartup{

	@Override
	public void earlyStartup() {
		Display.getDefault().syncExec(new Runnable(){
			@Override
			public void run() {		
				long eclipseStartTime = Long.parseLong(System.getProperty("eclipse.startTime"));
				long costTime = System.currentTimeMillis()-eclipseStartTime;
				Shell shell = Display.getDefault().getActiveShell();
				String message = "Eclipse启动耗时：" + costTime + "ms";
				MessageDialog.openInformation(shell, "Information", message);
			}
			
		});
	}
}

```

### 3) 修改plugin.xml文件如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<?eclipse version="3.4"?>
<plugin>
	<extension point="org.eclipse.ui.startup">
		<startup class="cn.hkjdy.showtime.ShowTime"/>
	</extension>
</plugin>
```

### 4) 运行插件

在项目上右键 -> Run As -> Eclipse Application，会启动Eclipse，启动完成后会显示Eclipse启动消耗时间，如下：

![JVM](/assets/eclipse-plug-in/03.png)

### 5) 导出插件

在项目上右键 -> Export... 在弹出的窗口中选择 `Deployable plug-ins and fragments` 如下：

![JVM](/assets/eclipse-plug-in/04.png)

在Destination中输入要导出的路径，如下：

![JVM](/assets/eclipse-plug-in/05.png)

导出成功后，插件会保存到导出路径的plugins文件夹下：ShowTime_1.0.0.201702211153.jar

## 二、Eclipse插件安装

把刚才导出的插件复制到eclipse目录的plugin目录下，然后启动eclipse，就可以看到eclipse启动消耗的时间。