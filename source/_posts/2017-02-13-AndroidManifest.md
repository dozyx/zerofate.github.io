---
title: AndroidManifest
tags:
  - android
date: 2017-02-13 15:37:22
categories: 笔记
---

[任务和返回栈](https://developer.android.google.cn/guide/components/tasks-and-back-stack.html)

[application-element](https://developer.android.com/guide/topics/manifest/application-element.html)

## \<application\>

### android:persistent

应用是否一直运行，默认false。应用通常不需要设置该属性，persistence模式只用于特定的系统应用。

## 属性

### android:taskAffinity

[android:taskAffinity](https://developer.android.com/guide/topics/manifest/activity-element.html#aff)

[Android–taskAffinity属性](http://www.androidchina.net/2649.html)

affinity是亲和的意思，该属性用来确定activity附属于哪个task，默认的affinity与包名一致。



### android:allowTaskReparenting

true时表示Activity可以从其启动的任务移动到与其具有关联的任务（如果该任务出现在前台）。

### android:usesCleartextTraffic

[Android M and the war on cleartext traffic](https://koz.io/android-m-and-the-war-on-cleartext-traffic/)

[从安全研究角度来看Android M（第一部分）](http://www.hackdig.com/07/hack-24372.htm)

默认为true，声明是否允许未加密的网络流量，如当usesCleartextTraffic被设置为false，应用程序会在使用HTTP而不是HTTPS时崩溃。

