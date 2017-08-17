---
title: dagger - CanReleaseReference 注解
tags:
  - dagger
date: 2017-08-08 15:37:22
categories: 笔记
---

[CanReleaseReferences](https://google.github.io/dagger/api/latest/dagger/releasablereferences/CanReleaseReferences.html)

```java
@Beta
 @Documented
 @Target(value=ANNOTATION_TYPE)
public @interface CanReleaseReferences
```

​	@Beta。用于注解 scope 注解，表明存储在作用域内引用的对象可能会在作用域的生命周期中释放。

​	 使用 scope 内可释放引用有两种方式，一是使用 @CanReleaseReference 对该注解进行注解，二是对该注解使用本身已被 @CanReleaseReference 注解的注解。

```java
 @Documented
    @Retention(RUNTIME)
    @CanReleaseReferences
    @Scope
   public  @interface MyScope {}
```

```java
@CanReleaseReferences
   public  @interface SomeAnnotation {}

    @Documented
    @Retention(RUNTIME)
    @SomeAnnotation
    @Scope
   public  @interface MyScope {}
```

