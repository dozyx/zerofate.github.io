---
title: 依赖注入
tags:
  - DI
date: 2017-08-07 15:37:22
categories: 笔记
---

[Dependency injection](https://en.wikipedia.org/wiki/Dependency_injection)

​	在软件工程中，依赖注入是一种对象提供对另一对象依赖关系的技术。 dependency 指的是将被使用的对象（service），injection 指的是将 dependency 传入到使用它的依赖对象（client）， service 将是 client 状态的一部分（个人理解是 service 是 client 的一个依赖，如果 service 变化，client 也会发生变化，即 client 的状态发生了变化）。**将 service 传递给 client，而不是允许 client 构建或者查找 service，这是该模式的基本要求。**

​	它的基本要求意味着禁止使用 new 或 static 方法在类中生成的值（service）(有点拗口，原文：This fundamental requirement means that using values (services) produced within the class from new or static methods is prohibited. )，该类应接受从外部传入的值。

​	**依赖注入的目的在于将对象解耦到不需要仅因为所依赖的对象更改为不同对象就更改 client 代码的程度。**

​	**依赖注入是控制反转（IOC，inversion of control）更广泛技术的一种形式。**高级代码接收它可以调用的更低级代码，而不是低级代码调用高级代码，这反转了在程序编码中经常看到的典型控制模式。

​	与其它形式的控制反转一样，**依赖注入支持依赖倒置原则**。client 将提供 dependency 的职责委托给了外部代码（injector），client 不允许调用 injector 的代码，注入的代码将构造 service 并调用 client 来对它们进行注入。这意味着 client 代码不需要了解注入的代码，不需要了解 service 是如何构建的，也不需要了解它正在使用哪些实际 service。client 只需要知道 service 的内在接口，因为它们定义了 client 如何使用这些 service。这就将使用和构造的职责进行了分离。 

​	client 接受依赖注入用三种常见的方法：setter、interface、constructor。setter 和 construtor 注入的差别主要在于何时被使用。interface 注入的不同之处在于dependency 有机会控制自己的注入。这些注入都要求独立的构造代码（injector）来负责引入 client 和彼此间的 dependency。



### Overview

​	依赖注入将 client 的依赖的创建与 client 的行为分开，这可以降低程序的耦合性，并遵循单一职责和依赖倒置原则。

​	依赖注入涉及了4个角色：

+ 使用的 **service** 对象
+ 依赖于使用的 service 的 **client** 对象
+ 定义了 client 如何使用 service 的 **interface**
+ 负责构造 service 和将它们注入 client 的 **injector**



### 例子

#### 无依赖注入

```
// An example without dependency injection
public class Client {
    // Internal reference to the service used by this client
    private ServiceExample service;

    // Constructor
    Client() {
        // Specify a specific implementation in the constructor instead of using dependency injection
        service = new ServiceExample();
    }

    // Method within this client that uses the services
    public String greet() {
        return "Hello " + service.getName();
    }
}
```



#### 依赖注入的三种类型

+ constructor 注入

  ```java
   // Constructor
  Client(Service service) {
      // Save the reference to the passed-in service inside this client
      this.service = service;
  }
  ```

+ setter 注入

  ```java
  // Setter method
  public void setService(Service service) {
      // Save the reference to the passed-in service inside this client
      this.service = service;
  }
  ```

+ interface 注入：dependency 提供一个 injector 方法，该方法将 dependency 注入到传递给它的任意 client 中。client 必须实现一个公开的接收 dependency 的 setter 方法的接口。

  ```Java
  // Service setter interface.
  public interface ServiceSetter {
      public void setService(Service service);
  }

  // Client class
  public class Client implements ServiceSetter {
      // Internal reference to the service used by this client.
      private Service service;

      // Set the service that this client is to use.
      @Override
      public void setService(Service service) {
          this.service = service;
      }
  }
  ```

  ​除了这三种类型，一些 DI 框架可能还会有其他类型的注入。如，一些现代测试框架甚至不要求 client 主动接受 dependency 注入，因为会造成可测试的遗留代码。特殊的，在 Java 中，可以使用反射在测试时将私有属性公开，从而通过赋值接受注入。

