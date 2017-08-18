---
title: Bean、POJO、EJB
tags:
  - java
date: 2017-03-19 15:37:22
categories: 笔记
---

[JavaBeans](https://docs.oracle.com/javase/tutorial/javabeans/)

[Java 之JavaBean 、EJB 和POJO](http://www.cnblogs.com/zhangminghui/p/4889761.html)

[What is a JavaBean exactly?](http://stackoverflow.com/questions/3295496/what-is-a-javabean-exactly)

[Java 帝国之Java bean (上）](https://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMzAxOTc0NzExNg%3D%3D%26mid%3D2665513115%26idx%3D1%26sn%3Dda30cf3d3f163d478748fcdf721b6414%23rd)

[Java 帝国之Java bean（下）](https://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMzAxOTc0NzExNg%3D%3D%26mid%3D2665513118%26idx%3D1%26sn%3D487fefb8fa7efd59de6f37043eb21799%23rd)

[Java bean 是个什么概念？](https://www.zhihu.com/question/19773379)

[POJO wiki](https://en.wikipedia.org/wiki/Plain_Old_Java_Object)

[Difference between DTO, VO, POJO, JavaBeans?](http://stackoverflow.com/questions/1612334/difference-between-dto-vo-pojo-javabeans)

# JavaBeans

> bean，豆子，本质是一个java类，而JavaBeans是bean所遵循的一套规范（编译工具如NetBeans可以进行解析）。

​	JavaBeans的目的是用来==使软件组件的复用变得更为简单==。开发者可以使用其他人写的软件组件而不需要理解其内部是如何工作的。

## 特点

+ 所有属性都是private（使用getters/setters访问）
+ 提供public的无参构造函数
+ 实现Serializable



## JavaBeans指南

​	bean编译工具(build tool，如NetBeans)可以计算出bean的属性、方法和事件。

### Properties

​	定义在bean类中的属性，应提供getter和setter方法。如：

```java
public class FaceBean {
    private int mMouthWidth = 90;

    public int getMouthWidth() {
        return mMouthWidth;
    }
    
    public void setMouthWidth(int mw) {
        mMouthWidth = mw;
    }
}
```

​	编译工具如NetBeans将理解这些方法名，并在属性列表中列出mounthWidth属性。

> 一个属性可能只有getter或者setter。

​	例外的，boolean属性允许使用is替代get访问方法。

#### 	带索引的属性（indexed properties）

```java
public int[] getTestGrades() {
    return mTestGrades;
}

public void setTestGrades(int[] tg) {
    mTestGrades = tg;
}
```

```java
public int getTestGrades(int index) {
    return mTestGrades[index];
}

public void setTestGrades(int index, int grade) {
    mTestGrades[index] = grade;
}
```

#### 	绑定属性（bound properties）

当值发送改变时通知监听器

+ 该bean类包含 addPropertyChangeListener()和removePropertyChangeListener()方法
+ 当bound property变化时，bean发送PropertyChangeEvent给已注册的监听器。

```java
import java.beans.*;

public class FaceBean {
    private int mMouthWidth = 90;
    private PropertyChangeSupport mPcs =
        new PropertyChangeSupport(this);

    public int getMouthWidth() {
        return mMouthWidth;
    }
    
    public void setMouthWidth(int mw) {
        int oldMouthWidth = mMouthWidth;
        mMouthWidth = mw;
        mPcs.firePropertyChange("mouthWidth",
                                   oldMouthWidth, mw);
    }

    public void
    addPropertyChangeListener(PropertyChangeListener listener) {
        mPcs.addPropertyChangeListener(listener);
    }
    
    public void
    removePropertyChangeListener(PropertyChangeListener listener) {
        mPcs.removePropertyChangeListener(listener);
    }
}
```

> bound properties可以通过编译工具如NetBeans直接与其它bean属性联系在一起。



#### 约束属性（constrained properties）

特殊的bound属性。对于constrained属性，bean将与一些veto（否决）监听器关联，当constrained属性要改变时，监听器对改变进行询问，任一监听器都有机会否决这些变化。

```java
import java.beans.*;

public class FaceBean {
    private int mMouthWidth = 90;
    private PropertyChangeSupport mPcs =
        new PropertyChangeSupport(this);
    private VetoableChangeSupport mVcs =
        new VetoableChangeSupport(this);

    public int getMouthWidth() {
        return mMouthWidth;
    }
    
    public void
    setMouthWidth(int mw) throws PropertyVetoException {
        int oldMouthWidth = mMouthWidth;
        mVcs.fireVetoableChange("mouthWidth",
                                    oldMouthWidth, mw);
        mMouthWidth = mw;
        mPcs.firePropertyChange("mouthWidth",
                                 oldMouthWidth, mw);
    }

    public void
    addPropertyChangeListener(PropertyChangeListener listener) {
        mPcs.addPropertyChangeListener(listener);
    }
    
    public void
    removePropertyChangeListener(PropertyChangeListener listener) {
        mPcs.removePropertyChangeListener(listener);
    }

    public void
    addVetoableChangeListener(VetoableChangeListener listener) {
        mVcs.addVetoableChangeListener(listener);
    }
    
    public void
    removeVetoableChangeListener(VetoableChangeListener listener) {
        mVcs.removeVetoableChangeListener(listener);
    }
}
```



### Methods

​	一个bean的method指它可以做的事（废话）。任何非属性定义部分的public方法都是一个bean方法。



### Events

​	一个bean类可以触发任何类型的event，包括自定义event。跟properties一样，events用特定形式的方法名来定义：

`public void add<Event>Listener(<Event>Listener a)`
`public void remove<Event>Listener(<Event>Listener a)`

​	listener类型必须是java.util.EventListener的子类。



## JavaBeans进阶

### Bean持久化（persistence）

​	所有的bean必须persist，因此，必须通过实现java.io.Serializable或者java.io.Externalizable来支持序列化。



### Long Term Persistence

​	使用xml形式保存bean。



### Bean Customization

​	*Customization* provides a means for modifying the appearance and behavior of a bean within an application builder so it meets your specific needs.



# POJO

​	plain old Java object，POJO是普通的Java对象，不被任何特定的限制约束，也没有任何class path的要求，与重量级的EJB相反。

>  	A JavaBean is a POJO that is serializable, has a no-argument constructor, and allows access to properties using getter and setter methods. An Enterprise JavaBean is not a single class but an entire component model (again, EJB 3 reduces the complexity of Enterprise JavaBeans).



# EJB

​	Enterprise JavaBeans







