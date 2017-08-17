---
title: dagger - Provides 注解
tags:
  - dagger
date: 2017-08-08 15:37:22
categories: 笔记
---

[Provides](https://google.github.io/dagger/api/latest/dagger/Provides.html)

> dagger 框架使用的注解

```java
@Documented 
@Target(METHOD) 
@Retention(RUNTIME)
public @interface Provides {
  enum Type {
    UNIQUE,
    SET,
    SET_VALUES,
    @Beta
    MAP;
  }

  Type type() default Type.UNIQUE;
}
```

​	用于注解 module （使用 @Module 注解的类）的方法，它将创建 provider 绑定方法，**该方法返回的 type 将绑定到其返回值。**习惯上，使用 @Provides 的方法名以 provide为前缀。

> 个人理解：对于一些无法直接使用 @Inject 的类，如第三方类，可以通过 @Provides 来实现注入，@Provides 注解的方法返回的值即实例将作为其方法返回类型所绑定的实例，即对该类型实现注入。

​	Type ：

+ UNIQUE：默认，此方法是给指定返回类型生成值的唯一方法。
+ SET：此方法的返回类型形成一个 Set\<T> 的通用类型参数，并且返回的值将赋予该 set。
+ SET_VALUES：