---
title: 什么是 instance initializer
tags:
  - Java
date: 2017-06-24 15:37:22
categories: 笔记
---

[What is Instance Initializer in Java?](http://www.programcreek.com/2011/10/java-class-instance-initializers/)

注意观察例子输出。

示例：

```java
public class Foo {
 
	//instance variable initializer
	String s = "abc";
 
	//constructor
	public Foo() {
		System.out.println("constructor called");
	}
 
	//static initializer
	static {
		System.out.println("static initializer called");
	}
 
	//instance initializer
	{
		System.out.println("instance initializer called");
	}
 
	public static void main(String[] args) {
		new Foo();
		new Foo();
	}
}
```

输出

```java
static initializer called
instance initializer called
constructor called
instance initializer called
constructor called
```

​	可以看到，instance initializer 是在构造函数执行之前进行的。

### 使用场景

​	instance initializer并不常见，一般可用于

+ 在有多个构造函数时，进行统一的初始化
+ 因为匿名内部类无法声明构造函数，所以可以在 Instance initializer 执行需要早构造前执行的代码。