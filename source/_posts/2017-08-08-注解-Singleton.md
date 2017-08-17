---
title: Singleton 注解
tags:
  - 注解
date: 2017-08-08 15:37:22
categories: 笔记
---

@Scope
@Documented
@Retention(RUNTIME)
public @interface Singleton {}

------

​	位于 javax.inject 包，标识 injector 只会对该类型实例化一次。此注解不会被遗传。