---
title: 设计模式 - 单例
tags:
  - 设计模式
date: 2016-11-02 15:37:22
categories: 笔记
---



## 从百度百科开始 ##

[百度百科](http://baike.baidu.com/link?url=2H9Dt1fniQr6bZoWZob2IICjQFO3okLzpBJ-mLNwE-jFeKISfTThiURSfgXsKFcXqtQqsKXwSgCBAPlZpJ_5FK)
### 定义 ###
最初的定义出现于《设计模式》（艾迪生维斯理，1994）：**保证一个类仅有一个实例，并提供一个访问它的全局访问点**

Java中单例模式定义：一个类有且仅有一个实例，并且自行实例化向整个系统提供。

### 目的

使得类的一个对象成为系统中的唯一实例

### 静态结构图

 ![单例模式的静态结构图](https://ws2.sinaimg.cn/large/006tKfTcgy1firs1xcbeyj306y02t74d.jpg)

### 要点

- 某个类只能有一个实例
- 必须自行创建这个实例
- 必须自行向整个系统提供这个实例

从具体实现角度：

- 单例模式的类只提供私有的构造函数
- 类定义中含有一个该类的静态私有对象
- 该类提供了一个静态的公有的函数用于创建或获取它本身的静态私有对象

### 实例

#### Java

一般Sigleton模式通常有三种形式：

第一种：懒汉型

```java
//线程不安全
public class SigletonClass {
  private static SigletonClass instance == null;
  public static synchronized SigletonClass getInstance(){
    if (instance == null){
      instance = new SigletonClass();
    }
    return instance;
  }
  private SigletonClass(){
  }
}
```

第二种：饿汉型

```java
//对第一行static的一些解释
//java允许我们在一个类里面定义静态类。比如内部类（nested class）。
//把nested class封闭起来的类叫外部类。
//在java中，我们不能用static修饰顶级类（top level class）。
//只有内部类可以为static。
public class Singleton{
    //在自己内部定义自己的一个实例，只供内部调用
    private static final Singleton instance = new Singleton();
    private Singleton(){
        //do something
    }
    //这里提供了一个供外部访问本class的静态方法，可以直接访问
    public static Singleton getInstance(){
        return instance;
    }
}
```

第三种：双重锁形式

```java
public class SingletonClass {
	private static SigletonClass instance = null;
	public static SigletonClass getInstance() {
		if(instance == null) {
			synchronized(SigletonClass.class) {
				if(instance == null) {
					instance = new SigletonClass();
				}
			}
		}
		return instance;
	private SigletonClass(){}
}
//这个模式将同步内容下方到if内部，提高了执行的效率，不必每次获取
//对象时都进行同步，只有第一次才同步，创建了以后就没必要了。
```



## 使用volatile写双重检验锁

[The "Double-Checked Locking is Broken" Declaration](http://www.cs.umd.edu/~pugh/java/memoryModel/DoubleCheckedLocking.html)

[关于java单例中使用使用volatile写双重检验锁的一点疑问？](https://www.zhihu.com/question/46903811)



## 疑问

### 单例是否可以使用参数

[Singleton with Arguments in Java](http://stackoverflow.com/questions/1050991/singleton-with-arguments-in-java)

在上面链接中，明确表示了“**a singleton with parameters is not a singleton**”的观点，如果需要在单例中传入参数，可以添加init方法或者在需要执行的方法中传入参数。其实这也很容易理解，单例的目的就是为了确保所有地方使用同一实例，如果每次getInstance时都使用了不同的参数，那么明显地单例将不再单例。

## 参考内容

[单例模式的七种写法](http://cantellow.iteye.com/blog/838473)（本文写得真好）

### 静态内部类

```java
//这种方式即使Singleton类被装载了，instance也不一定实例化
public class Singleton {  
    private static class SingletonHolder {  
    private static final Singleton INSTANCE = new Singleton();  
    }  
    private Singleton (){}  
    public static final Singleton getInstance() {  
    return SingletonHolder.INSTANCE;  
    }  
}  
```

### 枚举

```
//该方式为Effictive Java作者Josh Bloch提倡的方式
//不仅能避免多线程同步问题，而且还能反序列化重新创建新的对象
public enum Singleton {  
    INSTANCE;  
    public void whateverMethod() {  
    }  
}  
```

## 资料阅读

### Lazy loading（延迟加载）

[维基](https://en.wikipedia.org/wiki/Lazy_loading)

lazy loading是一种在计算机编程语言中通常用于对一个对象进行延迟实例化直到被需要的一种设计模式。恰当的使用有助于提高程序的运行效率，其对立为eager loading。

有四种常用的lazy load设计模式实现方法，每一种都有其优缺点：

#### [lazy initialization](https://en.wikipedia.org/wiki/Lazy_initialization)

在计算机编程语言中，lazy initialization目的在于延迟对象的创建、值的计算或者其他昂贵的处理，直到第一次使用。lazy initialization通常与工厂模式（lazy factory）一起使用。

示例：

```
import java.util.HashMap;
import java.util.Map;
import java.util.Map.Entry;

public class Program {

    /**
     * @param args
     */
    public static void main(String[] args) {
        Fruit.getFruitByTypeName(FruitType.banana);
        Fruit.showAll();
        Fruit.getFruitByTypeName(FruitType.apple);
        Fruit.showAll();
        Fruit.getFruitByTypeName(FruitType.banana);
        Fruit.showAll();
    }
}

enum FruitType {
    none,
    apple,
    banana,
}

class Fruit {

    private static Map<FruitType, Fruit> types = new HashMap<>();
    
    /**
     * Using a private constructor to force the use of the factory method.
     * @param type
     */
    private Fruit(FruitType type) {
    }
    
    /**
     * Lazy Factory method, gets the Fruit instance associated with a certain
     * type. Instantiates new ones as needed.
     * @param type Any allowed fruit type, e.g. APPLE
     * @return The Fruit instance associated with that type.
     */
    public static Fruit getFruitByTypeName(FruitType type) {
        Fruit fruit;
                // This has concurrency issues.  Here the read to types is not synchronized, 
                // so types.put and types.containsKey might be called at the same time.
                // Don't be surprised if the data is corrupted.
        if (!types.containsKey(type)) {
            // Lazy initialisation
            fruit = new Fruit(type);
            types.put(type, fruit);
        } else {
            // OK, it's available currently
            fruit = types.get(type);
        }
        
        return fruit;
    }
    
    /**
     * Lazy Factory method, gets the Fruit instance associated with a certain
     * type. Instantiates new ones as needed. Uses double-checked locking 
     * pattern for using in highly concurrent environments.
     * @param type Any allowed fruit type, e.g. APPLE
     * @return The Fruit instance associated with that type.
     */
    public static Fruit getFruitByTypeNameHighConcurrentVersion(FruitType type) {
        if (!types.containsKey(type)) {
            synchronized (types) {
                // Check again, after having acquired the lock to make sure
                // the instance was not created meanwhile by another thread
                if (!types.containsKey(type)) {
                    // Lazy initialisation
                    types.put(type, new Fruit(type));
                }
            }
        }
        
        return types.get(type);
    }
    
    /**
     * Displays all entered fruits.
     */
    public static void showAll() {
        if (types.size() > 0) {
 
           System.out.println("Number of instances made = " + types.size());
            
            for (Entry<FruitType, Fruit> entry : types.entrySet()) {
                String fruit = entry.getKey().toString();
                fruit = Character.toUpperCase(fruit.charAt(0)) + fruit.substring(1);
                System.out.println(fruit);
            }
            
            System.out.println();
        }
    }
}
```

#### Virtual proxy

virtual proxy是具有与真实对象相同接口的一个对象。在它的方法被第一次调用时，加载真实对象并进行代理。

#### Ghost

一个"ghost"是一个将在局部状态加载的对象。

#### Value holder

value holder是一个处理lazy loading行为的通常对象，用于代替对象的数据字段。





















