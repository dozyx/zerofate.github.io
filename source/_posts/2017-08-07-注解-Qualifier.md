---
title: Qualifier注解
tags:
  - 注解
date: 2017-08-07 15:37:22
categories: 笔记
---

[Qualifier](http://docs.oracle.com/javaee/7/api/javax/inject/Qualifier.html)

```java
@Target(ANNOTATION_TYPE)
@Retention(RUNTIME)
@Documented
public @interface Qualifier {}
```

​	标识限定符（qualifier）注解，任何人都可以定义一个新的 qualifier。

​	qualifier 注解：

+ 与 @Qualifier、@Retention(RUNTIME)，通常还有 @Documented 一起进行注解
+ 可以有属性
+ 可能是公共 API 的一部分，非常像 dependency 类型，但不同于不需要作为公共 API 一部分的实现类型（？）

> 关于 @Qualifier 还不是很理解，好像可以用于依赖注入框架用来限定可注入字段使用的具体实现。等以后见到更多实际使用后，再做补充。
>
> 个人理解：@Qualifier 是一个元注解，用于创建一个限定符注解，然后使用该注解来限定某个元素。

例子：

```java
   @java.lang.annotation.Documented
   @java.lang.annotation.Retention(RUNTIME)
   @javax.inject.Qualifier
   public @interface Leather {
     Color color() default Color.TAN;
     public enum Color { RED, BLACK, TAN }
   }
```