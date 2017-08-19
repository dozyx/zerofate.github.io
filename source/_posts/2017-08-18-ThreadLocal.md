---
title: ThreadLocal 理解
tags:
  - java
date: 2017-08-18 23:58:56
categories: Java
---

> 先在这里给出自己对 ThreadLocal 的理解：
>
> ThreadLocal 是线程安全的，它解决的是**同一线程**内“共享”数据的问题。在使用时，多个线程共享同一个 ThreadLocal 实例，但每个线程拥有自己单独的 ThreadLocal 值（调用 get 获取，set 设置），即一个线程对值的修改不会影响其他线程的值。 ThreadLocal 在使用时通常需要重写 initialValue() 方法，在 get 时如果 ThreadLocal 的值没有初始化，将回调该方法。ThreadLocal 变量通常声明为私有静态，它的状态与线程有关（即 ThreadLocal 的值与线程自身有关）。
>
> 文章中涉及的英文翻译可能不太好，附上ThreadLocal 的  Javadoc 原文：
>
> This class provides thread-local variables. These variables differ from their normal counterparts in that each thread that accesses one (via its `get` or `set` method) has its own, independently initialized copy of the variable. `ThreadLocal` instances are typically private static fields in classes that wish to associate state with a thread (e.g., a user ID or Transaction ID).



查看 Javadoc，第一句「提供了线程本地变量」，嗯哈。。。不懂。。。接着看，「这些变量与它们普通变量的区别在于，每个通过 get 或 set 方法访问它们的线程都会拥有独立初始化的变量副本。ThreadLocal 实例通常是类中的私有静态字段，以使其状态与一个线程(如用户 ID 或事物 ID)关联。」。。。还是不懂。。

看个例子吧，下面的类将为每个线程生成唯一的本地标识符，线程的 ID 将在第一次调用 ThreadId.get() 时被分配，并在后续调用中保持不变。

```java
 import java.util.concurrent.atomic.AtomicInteger;

 public class ThreadId {
     // Atomic integer containing the next thread ID to be assigned
     private static final AtomicInteger nextId = new AtomicInteger(0);

     // Thread local variable containing each thread's ID
     private static final ThreadLocal<Integer> threadId =
         new ThreadLocal<Integer>() {
             @Override protected Integer initialValue() {
                 return nextId.getAndIncrement();
         }
     };

     // Returns the current thread's unique ID, assigning it if necessary
     public static int get() {
         return threadId.get();
     }
 }
```

「只要线程存活并且 ThreadLocal 实例可访问，每个线程都会持有一个对自身 thread-local 变量副本的引用；线程消失后，thread-local 实例的所有副本都将被垃圾回收，除非存在对这些副本的其他引用。」

为了验证 ThreadId 的功能，我写了下面这个类

```java
class TestRun {
	public static void main(String[] args) {
		new Thread(new Runnable() {
			public void run() {
				System.out.println(new ThreadId().get());
			}
		}).start();
		try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
		new Thread(new Runnable() {
			public void run() {
				System.out.println(new ThreadId().get());
				System.out.println(new ThreadId().get());
			}
		}).start();
		try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
		new Thread(new Runnable() {
			public void run() {
				System.out.println(new ThreadId().get());
			}
		}).start();
	}
}
```

运行输出结果为「0，1，1，2」。从结果来看，大致可以得到结论——每一个线程所取得的 ThreadLocal 实例都是独立的。为了验证这个结论，我在 ThreadId 类中加了一个 set 方法来调用 threadId.set()，然后在 TestRun 的第一个线程执行中使用 set 将 threadId 的值改变，但后面的线程的值并没有变化，这刚好验证了前面的结论。

在 ThreadId 类的 initialValue() 方法中加上打印，可以看到，该方法只在第一次调用 get 时才会调用。

经过上面的分析，可以得到以下结论：

+ 在第一次调用 ThreadLocal 的 get 方法时，将调用 initialValue() 对 ThreadLocal 的值进行初始化。其实，这里说得不严谨，因为，假如在 get 之前调用了 set 方法，那么 ThreadLocal 值得初始化将在 set 时完成，set 后调用的 get 不会导致 initialValue() 方法的调用。
+ 每个线程将单独拥有一份 ThreadLocal 的值，即一个线程会对应一个值。这样，一个线程对值的改变不会影响其他线程。



参考：

[ThreadLocal](https://developer.android.com/reference/java/lang/ThreadLocal.html)

[ThreadLocal 源码](https://android.googlesource.com/platform/libcore/+/refs/heads/master/ojluni/src/main/java/java/lang/ThreadLocal.java)

[深入深入理解ThreadLocal](http://www.cnblogs.com/lqminn/p/3751206.html)

[谈谈ThreadLocal和解决线程安全的关系](http://zhangbo-peipei-163-com.iteye.com/blog/2029216?utm_source=qq&utm_medium=social)



