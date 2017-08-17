---
title: dagger - Reusable 注解
tags:
  - dagger
date: 2017-08-14 15:37:22
categories: 笔记
---

[Reusable](https://google.github.io/dagger/api/latest/dagger/Reusable.html)

```java
@Documented
 @Beta
 @Retention(value=RUNTIME)
 @Scope
public @interface Reusable
```

> ​	@Beta，处于测试中的注解，暂时做下记录。

​	一个 scope 注解，表明一次 binding 中返回的对象可能被重复使用，也可能不会。@Reusable 用于需要限制一个 type 的实例的数量，但又不需要确保其唯一性的情况。