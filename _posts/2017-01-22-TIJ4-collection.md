---
date: 2017/1/22 星期日  18:30:51+0800
layout: post
title: Java编程思想-集合类
thread: 164
categories: 读书笔记
tags:  Java编程思想
---

## 集合类

Collection、List、Set、Queue、Map。

## 基本概念

Java容器类库的用途是**保存对象**，并将其划分为两个不同的概念：

1. Collection

一个独立元素的序列，这些元素都服从一条或多条规则。List按照插入的顺序保存元素，Set不能有重复元素，Queue按照排队规则来确定对象产生的顺序。

2. Map

一组成对的**键值对**对象，允许你使用键来查找值。**ArrayList**允许你使用数字来查找值，**映射表**允许我们使用另一个对象来查找某个对象，也被称为**关联数组**或**字典**。

## List

以特定的顺序保存一组元素。

```java
public static void main(String[] args) {
	Random random = new Random(47);
	List<Integer> list = new ArrayList<Integer>();
	for(int i=0; i<10; i++) {
		list.add(random.nextInt(20));
	}
	System.out.println(list);
}
```

输出

```
[18, 15, 13, 1, 1, 9, 8, 0, 2, 7]
```

### 子类

#### ArrayList

- 随机访问速度快

- 插入和移动元素慢

#### LinkedList

- 插入和删除操作速度快

- 随机访问速度慢

### 类图

![list](/assets/collection/list.png)

<a href="http://www.hkjdy.cn/assets/collection/list.png" target="_blank">查看大图</a>


## Map

在每个槽内保存了两个对象，即键和与之相关联的值。

```java
public static void main(String[] args) {
	Random random = new Random(47);
	Map<Integer, String> map = new HashMap<Integer, String>();
	for(int i=0; i<10; i++) {
		map.put(random.nextInt(20), ""+random.nextInt(20));
	}
	System.out.println(map);
}
```

输出

```
{0=18, 1=9, 18=1, 2=7, 8=8, 9=18, 11=9, 13=1}
```

### 子类

#### HashMap

- Map 基于散列表的实现

- 设计用来快速访问

- 保存数据无顺序

- 插入和查询的开销是固定的

#### TreeMap

- 基于红黑树的实现

- 按比较结果升序保存键

- 保持**键**始终处于排序状态

- 没有HashMap 速度快

- 唯一带有subMap()方法的Map

#### LinkedHashMap

- 按照插入顺序或基于由于访问的 **最近最少使用（LRU）**算法保存键

- 只比HashMap慢一点

- 迭代速度比HashMap快，因为它使用链表维护内部次序

#### WeakHashMap

- 弱键映射，允许释放映射所指向的对象

- 是为解决某类特殊问题而设计的

- 如果映射这外没有引用指向某个**键**，则此**键**可以被垃圾收集器回收

#### CurrentHashMap

- 线程安全的Map，它不涉及同步加锁

#### IndentityHashMap

- 使用==代替equals()对**键**进行比较的散列映射

### 操作

#### 清空数据

```java
map.keySet().removeAll(map.keySet())
```

或者

```java
map.clear()
```

### 散列与散列码

- 如果使用自己的类做为HashMap的键，必须同时重载hashCode()和equals()

- 正确的equals()方法必须满足下列5个条件

自反性、对称性、传递性、一致性、对任何不是null的x，x.equals(null)一定返回是false

- 为使用散列均匀，桶的数量通常使用质数。但是对现代处理器来说桶的数量使用2的整数次方速度更快。

- 怎么写一份像样的hashCode()

1. 给int变量赋予某个非零值常

2. 为对象内每个有意义的域f（即每个可做equals()操作的域）计算出一个int散列码c

3. 合并计算得到的散列码

4. 返回result

5. 检查hashCode()最后生成的结果要，确保相同的对象有相同的散列码

### 类图

![map](/assets/collection/map.png)

<a href="http://www.hkjdy.cn/assets/collection/map.png" target="_blank">查看大图</a>

## Set

元素不能重复。

```java
public static void main(String[] args) {
	Random random = new Random(47);
	Set<Integer> set = new HashSet<Integer>();
	for(int i=0; i<10; i++) {
		set.add(random.nextInt(100));
	}
	System.out.println(set);
	
}
```

输出

```
[0, 68, 22, 55, 7, 58, 93, 61, 29]
```

### 子类

#### HashSet

- 如果没有其它的限制，应该是首选Map

- 提供最快的查询速度

- 元素必须定义hashCode()

- 使用散列函数


#### TreeSet

- 保持元素处于排序状态

- 元素必须实现Comparable接口

- 元素存储在红-黑树数据结构中

- 迭代通常比HashSet快

#### LinkedHashSet

- 具有HashSet的查询速度

- 以插入顺序保存元素

- 使用链表维护元素的顺序

- 元素必须定义hashCode()方法

### 类图

![set](/assets/collection/set.png)

<a href="http://www.hkjdy.cn/assets/collection/set.png" target="_blank">查看大图</a>


## Queue

队列是一个典型的**先进先出（FIFO）**容器。即从容器的一端放入事物，从另一端取出，并且事物放入容器的顺序与取出的顺序是相同的。

LinkedList提供了方法以支持队列的行为，并且它实现了Queue接口，因此LinkedList可以用作Queue的一种实现。

```java
public static void main(String[] args) {
	Random random = new Random(47);
	Queue<Integer> queue = new LinkedList<Integer>();

	for(int i=0; i<10; i++) {
		queue.offer(random.nextInt(20));
	}
	
	System.out.println("queue: " + queue);
	
	System.out.println("queue.peek(): " + queue.peek());

	System.out.println("queue.element():" + queue.element());
	
	System.out.println("queue.poll(): " + queue.poll());
	
	System.out.println("queue: " + queue);
	
	System.out.println("queue.remove(): " + queue.remove());
	
	System.out.println("queue: " + queue);
	
	
}
```

输出

```
queue: [18, 15, 13, 1, 1, 9, 8, 0, 2, 7]
queue.peek(): 18
queue.element():18
queue.poll(): 18
queue: [15, 13, 1, 1, 9, 8, 0, 2, 7]
queue.remove(): 15
queue: [13, 1, 1, 9, 8, 0, 2, 7]
```

offer()方法是与Queue相关的方法之一，它在允许的情况下，将一个元素插入到队尾，或者返回false。

peek()和element()都在不移除的情况下返回队头，但是peek()方法在队列为空时返回null，而element()会抛出**java.util.NoSuchElementException**异常.

poll()和remove()方法将移除并返回队头，但是poll()在队列为空时返回null，而remove()会抛出**java.util.NoSuchElementException**异常。

### 子类

#### PriorityQueue

优先级队列声明下一个弹出元素是最需要的元素（具有最高的优先级）。

```java
public static void main(String[] args) {
	Random random = new Random(47);
	
	PriorityQueue<Integer> queue = new PriorityQueue<Integer>();
	
	for(int i=0; i<10; i++) {
		queue.offer(random.nextInt(i +10));
	}
	
	System.out.println("queue:" + queue);
	
	for(int i=0; i<10; i++) {
		System.out.print(queue.poll() + " ");
	}
	
	System.out.println();
}
```

输出

```
queue:[0, 1, 1, 1, 1, 14, 3, 8, 1, 5]
0 1 1 1 1 1 3 5 8 14 
```

PriorityQueue可以确保当你调用peek()、poll()和remove()方法时，获取的元素将是队列中优先级最高的元素。默认的排序将使用对象在队列中的自然排序，但是可以通过提供自己的Comparator来修改这个顺序。


### 类图

![queue](/assets/collection/queue.png)

<a href="http://www.hkjdy.cn/assets/collection/queue.png" target="_blank">查看大图</a>

## 迭代器

迭代器是一个对象，它的工作是遍历并选择序列中的对象，而客户端程序不必知道或关心该序列底层的结构。此外迭代器通常被称为**轻量级对象**：创建它的代价小。因此，经常可以见到对迭代器有些奇怪的限制；例如，Java的Iterator只能单向移动，这个Iterator只能用来：

1. 使用方法iterator()要求容器返回一个Iterator。Iterator准备好返回序列的第一个元素。

2. 使用next()获得序列中的下一个元素。

3. 使用hasNext()检查序列中是否还有元素。

4. 使用remove()将迭代器新近返回的元素删除。

### 操作

```java
public static void main(String[] args) {
	Random random = new Random(47);
	List<Integer> list = new ArrayList<Integer>();
	for(int i=0; i<10; i++) {
		list.add(random.nextInt(20));
	}
	Iterator<Integer> it = list.iterator();
	while(it.hasNext()) {
		System.out.print(it.next() + " ");
	}
	System.out.println();
	
	it = list.iterator();
	while(it.hasNext()) {
		if(it.next() == 1) {
			it.remove();
		}
	}
	
	System.out.println(list);
}
```

输出

```
18 15 13 1 1 9 8 0 2 7 
[18, 15, 13, 9, 8, 0, 2, 7]
```

### ListIterator

ListIterator是一个更加强大的Iterator子类型，它只能用于各种List类的访问。尽管Iterator只能向前移动，但是ListIterator可以双向移动。它还可以产生相对于迭代器在列表中指向的当前位置的前一个和后一个元素的索引，并且可以使用set()方法替换它访问过的最后一个元素。你可以通过调用listIterator()方法产生一个指向List开始处的ListIterator，并且还可以通过调用listIterator(n)方法创建一个一开始就指向列表索引为n的元素处的ListIterator。下面的示例演示了这些能力：

```java
public static void main(String[] args) {
	Random random = new Random(47);
	List<Integer> list = new ArrayList<Integer>();
	for(int i=0; i<10; i++) {
		list.add(random.nextInt(20));
	}
	ListIterator<Integer> iterator = list.listIterator();
	while(iterator.hasNext()) {
		System.out.print(iterator.next() + " ");
	}
	System.out.println();
	
	iterator = list.listIterator(3);
	while(iterator.hasNext()) {
		System.out.print(iterator.next() + " ");
	}
	System.out.println();
	
	iterator = list.listIterator(list.size());
	while(iterator.hasPrevious()) {
		System.out.print(iterator.previous() + " ");
	}
	System.out.println();
}

```

输出

```
18 15 13 1 1 9 8 0 2 7 
1 1 9 8 0 2 7 
7 2 0 8 9 1 1 13 15 18 
```

## Foreach

foreach语法除了用于数组以外，还可以应用于任何Collection对象。

```java
public static void main(String[] args) {
	Random random = new Random(47);
	List<Integer> list = new ArrayList<Integer>();
	for(int i=0; i<10; i++) {
		list.add(random.nextInt(20));
	}
	for(Integer i : list) {
		System.out.print(i+" ");
	}
	System.out.println();
}
```

输出

```
18 15 13 1 1 9 8 0 2 7 
```