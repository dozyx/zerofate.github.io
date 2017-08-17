---
title: Retention 注解
tags:
  - 注解
date: 2017-07-20 15:37:22
categories: 笔记
---

[Retention](https://developer.android.com/reference/java/lang/annotation/Retention.html)

[RetentionPolicy](https://developer.android.com/reference/java/lang/annotation/RetentionPolicy.html) 

[Java注解之Retention、Documented、Inherited介绍](http://www.jb51.net/article/55371.htm)

public abstract @interface Retention 
implements Annotation

java.lang.annotation.Retention

------

​	表明带注解类型的注解应该保留到哪个阶段，如果一个注解类型在声明时没有使用 Retention 注解，将使用默认的RetentionPolicy.CLASS 策略。

​	RetentionPolicy 为 enum 类型，它定义了三个常量，即三种策略：

+ SOURCE

  注解将由编译器废弃，即只保留在源代码级别，编译时会忽略。

+ CLASS

  注解将被编译器记录在 class 文件中，但 VM 在运行时不会保留。CLASS 是**默认**的值。

+ RUNTIME

  注解将被编译器记录在 class 文件中，并且 VM 在运行时将保留。因此，它们**可以在运行时通过反射读取**。