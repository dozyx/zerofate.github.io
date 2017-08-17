---
title: dagger - Lazy 接口
tags:
  - dagger
date: 2017-08-14 15:37:22
categories: 笔记
---

[Lazy](https://google.github.io/dagger/api/latest/dagger/Lazy.html)

```java
public interface Lazy<T> {
  T get();
}
```

> 注意 Lazy 只是个普通的 interface。

​	Lazy 只在第一次调用 get() 时才会计算它的值并为随后的 get() 记忆同一个值。



### 例子

​	直接注入、provider 注入和 lazy 注入的区别可以使用下面的例子来说明。 下面的 module 为每一次使用计算一个不同的整型。

```java
    @Module
   final class CounterModule {
     int next = 100;

      @Provides Integer provideInteger() {
       System.out.println("computing...");
       return next++;
     }
   }
```

**直接注入**

```java
   final class DirectCounter {
      @Inject Integer value;

     void print() {
       System.out.println("printing...");
       System.out.println(value);
       System.out.println(value);
       System.out.println(value);
     }
   }
```

​	将输出：

   computing...
   printing...
   100
   100
   100

**provider 注入**

```java
   final class ProviderCounter {
      @Inject Provider<Integer> provider;

     void print() {
       System.out.println("printing...");
       System.out.println(provider.get());
       System.out.println(provider.get());
       System.out.println(provider.get());
     }
   }

```

​	输出：

   printing...
   computing...
   100
   computing...
   101
   computing...
   102



**lazy 注入**

```java
   final class LazyCounter {
      @Inject Lazy<Integer> lazy;

     void print() {
       System.out.println("printing...");
       System.out.println(lazy.get());
       System.out.println(lazy.get());
       System.out.println(lazy.get());
     }
   }
```

​	输出：

   printing...
   computing...
   100
   100
   100