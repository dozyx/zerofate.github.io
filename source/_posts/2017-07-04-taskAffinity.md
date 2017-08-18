---
title: Activity 的 taskAffinity 属性
tags:
  - android
date: 2017-07-04 15:37:22
categories: 笔记
---

[android:taskAffinity](https://developer.android.com/guide/topics/manifest/activity-element.html#aff)

[Android–taskAffinity属性](http://www.androidchina.net/2649.html)

​	affinity，亲和性。该属性用于指定一个对该 activity 具有亲和性的 task。具有相同 affinity 的 activity 归属于同一个 task。一个 task 的affinity 由它的根 activity 确定。

​	affinity 决定了两件事：

+ activity 可以 re-parent 的 task (android:allowTaskReparenting 属性为true时，如果相同 affinity 的 task 进入前台，activity 将重新依附)
+ 以 FLAG_ACTIVITY_NEW_TASK 启动的 activity 该附属的 task，如果该 task 存在则添加到该 task，不存在则创建该 task

  ​默认地，一个应用中所有的 activity 有相同的 affinity。我们可以设置 android:taskAffinity 来对它们进行分组，甚至将不同应用里的 activity 放在同一个 task。如果需要声明一个 activity 对任何 task 都没有 affinity，可以将 affinity 设置为一个空的字符串。

  ​如果 android:taskAffinity 没有设置，activity 会从 \<application> 中继承。一个应用默认的 affinity 名称是 \<manifest> 中设置的包名。