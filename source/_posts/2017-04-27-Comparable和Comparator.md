---
title: Comparable 和 Comparator
tags:
  - Java
date: 2017-04-27 15:37:22
categories: 笔记
---

 [java比较器Comparable接口和Comaprator接口](http://blog.csdn.net/zhushuai1221/article/details/51760663)

### Comparable

```java
public interface Comparable<T> {
    int compareTo(T another);
}
```

用于对某个类的实例进行排序。Collections.sort 和 Arrays.sort 可用来对实现了该接口的类进行自动排序。

compareTo 返回值：

+ 负数：该实例小于 another
+ 正数：该实例大于 another
+ 0：该实例与 another 排序一样




### Comparator

```java
public interface Comparator<T> {
    public int compare(T lhs, T rhs);
    public boolean equals(Object object);
}
```

​	Comparator 用于比较两个对象以决定相互间的顺序。

compare 返回值：

+ 负数：lhs 小于 rhs
+ 正数：lhs 大于 rhs
+ 0： 相等





### Comparable 和 Comparator 的区别

​	Comparable 的比较方法接受一个参数，该接口由需要进行比较的类自身进行实现；而Comparator 的比较方法接受两个参数，其用法通常是创建一个专用的Comparator 子类，然后在进行sort操作时传递进去。Comparator 解决了类在设计时没有实现 Comparable 接口进行比较的问题，又或者通过实现自定义的Comparator解决类自身的Comparable 无法满足需要的问题。

​	实现上，两种比较器的作用并没有什么区别。