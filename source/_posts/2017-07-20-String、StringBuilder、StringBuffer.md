---
title: String、StringBuilder、StringBuffer 的区别
tags:
  - java
date: 2017-07-20 15:37:22
categories: 笔记
---

[String](https://developer.android.com/reference/java/lang/String.html)

[StringBuffer](https://developer.android.com/reference/java/lang/StringBuffer.html)

[StringBuilder](https://developer.android.com/reference/java/lang/StringBuilder.html)

《Java:突破程序员基本功的16课》——关于字符串的陷阱

​	String 代表字符序列不可变的字符串，相当于常量，而 StringBuilder 和 StringBuffer 都代表字符序列可变的字符串。如果程序使用多个 String 对象进行字符串连接运算，在运行时将生成大量临时字符串，这些字符串会保存在内存中从而导致程序性能下降。



+ **StringBuffer**

  **线程安全**的可变字符序列，它使用 synchronized 来确保执行操作的方法串行执行。

+ **StringBuilder**

  StringBuffer 的非同步版本，同样是用于表示可变字符序列，但执行效率比 StringBuffer 高。如果不考虑多线程，应尽量使用 StringBuilder。