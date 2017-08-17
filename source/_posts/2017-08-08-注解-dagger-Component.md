---
title: dagger - Component
tags:
  - dagger
date: 2017-08-08 15:37:22
categories: 笔记
---

[Component](https://google.github.io/dagger/api/latest/dagger/Component.html)



```java
@Retention(RUNTIME)
@Target(TYPE)
@Documented
public @interface Component {
  Class<?>[] modules() default {};
  Class<?>[] dependencies() default {};
  
  @Target(TYPE)
  @Documented
  @interface Builder {}
}

```

​	注解接口或抽象类，将**从一组 module （指用 @Module 注解的类）中生成一个完全形成的依赖注入实现类**。生成类的名称将带有 Dagger 前缀，如，`@Component interface  MyComponent{...}` 将生成一个名为 DaggerMyComponent。



### Component 方法

​	**每一个使用 @Component 注解的 type 必须包含至少一个抽象 component 方法。**component 方法可以有任意的名称，但其签名必须符合  [provision](https://docs.oracle.com/javaee/7/api/javax/inject/Provider.html?is-external=true)（对应 Provider\<T> 接口，提供一个 T 的实例） 或者 [members-injection](https://google.github.io/dagger/api/latest/dagger/MembersInjector.html) （对应 MembersInjector\<T> 接口，将 dependency 注入实例的字段和方法中）规定。

> provision，供应，向外部提供值；members-injection，外部向内部传入实例，然后将注解的字段和方法注入到该实例。

#### provision 方法

​	provision 方法没有参数并返回一个 injected （对应 @Inject）或 provided （对应 @Provides）的 type。如：

​		SomeType getSomeType(); 

​		Set\<SomeType> getSomeTypes(); 

​		@PortNumber int getPortNumber();



#### Members-injection 方法

​	member-injection 方法具有单个参数，并将 dependency 注入到所传入实例的每一个 @Inject 注解的字段和方法中。

​	如下均为有限的 member-injection 方法声明：

​		void injectSomeType(SomeType someType);
​		SomeType injectAndReturnSomeType(SomeType someType);

​	一个没有参数并返回 MembersInjector 的方法等同于 member injection 方法。如：

​		MembersInjector\<SomeType> getSomeTypeMembersInjector();



​	