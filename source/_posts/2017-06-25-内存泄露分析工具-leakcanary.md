---
title: 内存泄露分析工具LeakCanary
tags:
  - android
  - 内存泄露
date: 2017-06-25 15:37:22
categories: 笔记
---

[leakcanary github](https://github.com/square/leakcanary)

[Why should I use LeakCanary?](https://github.com/square/leakcanary/wiki/FAQ#why-should-i-use-leakcanary)

​	leakcanary(canary，金丝雀)是一个用于侦测 Android 和 Java 库中内存泄露的库。在发现 activity 内存泄露时，将会发出通知，查看通知可以直接显示出发生泄露的代码。（下面的泄露原因是后台执行任务未结束时旋转屏幕）

![leakcanary](https://ws4.sinaimg.cn/large/006tKfTcgy1finu7793p1j30jg0a4aat.jpg)

### 使用

1. 在 build.gradle 中添加依赖

   根据不同的 compile，将使用不同的 leakcanary 版本。其中，no-op 版本只包含了LeakCanary和RefWatcher类，相当于空依赖，这样才能使编译正常通过。

   ```groovy
    dependencies {
      debugCompile 'com.squareup.leakcanary:leakcanary-android:1.5.1'
      releaseCompile 'com.squareup.leakcanary:leakcanary-android-no-op:1.5.1'
      testCompile 'com.squareup.leakcanary:leakcanary-android-no-op:1.5.1'
    }
   ```


2. 在自定义 Application 中配置，代码如：

   ```Java
   public class ExampleApplication extends Application {

     @Override public void onCreate() {
       super.onCreate();
       if (LeakCanary.isInAnalyzerProcess(this)) {
         // This process is dedicated to LeakCanary for heap analysis.
         // You should not init your app in this process.
         return;
       }
       LeakCanary.install(this);
       // Normal app init code...
     }
   }
   ```

配置完后，如在在debug 编译中出现 **activity 内存泄露**，LeakCanary 将自动显示通知。



### LeakCanary 定制

[Customizing LeakCanary](https://github.com/square/leakcanary/wiki/Customizing-LeakCanary)

​	在应用的 release 版本中使用的是leakcanary-android-no-op，该版本的 leakcanary 只有LeakCanary和RefWatcher类。如果需要定制 LeakCanary，则需要确保定制只在 debug 编译版本中使用，否则可能会引用leakcanary-android-no-op中不存在的类。

通常的定制化有：

+ 自定义 icon 和 label
+ leak traces 保存
+ 上传到服务器
+ 忽略特定引用
+ 不监视指定的 Activity 类
+ 运行时触发 LeakCanary 开/关