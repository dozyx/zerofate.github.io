---
title: dagger - Module注解
tags:
  - dagger
date: 2017-08-08 15:37:22
categories: 笔记
---

[Module](https://google.github.io/dagger/api/latest/dagger/Module.html)

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface Module {
  /**
   * Additional {@code @Module}-annotated classes from which this module is
   * composed. The de-duplicated contributions of the modules in
   * {@code includes}, and of their inclusions recursively, are all contributed
   * to the object graph.
   */
  Class<?>[] includes() default {};
}
```

​	注解有助于 object graph （当前应用的实例网络）的类。



#### includes

​	组成该 module 的其它由 @Module 注解的类。