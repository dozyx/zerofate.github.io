---
title: Activity 的 launchMode 属性
tags:
  - android
date: 2016-11-09 15:37:22
categories: 笔记
---

[深入讲解Android中Activity launchMode](http://droidyue.com/blog/2015/08/16/dive-into-android-activity-launchmode/index.html)

[Understand Android Activity's launchMode](https://inthecheesefactory.com/blog/understand-android-activity-launchmode/en)

> 上面参考文章中提到 Android 5.0+ 从一个应用启动另一个应用的 Activity 会放入另一个 task，但评论中有人指出实际结果并不符合。

​	StandardActivity每一次启动时都会进行新建；SingleTopActivity处于task顶端时，不会进行新建，反之，则会；SingleTaskActivity只会存在一个，再次启动时，将清除掉处于它上面的activity；SingleInstanceActivity同样只会存在一个，并且不会影响到其他activity所在task的顺序，如先启动了SingleTopActivity，接着启动SingleInstanceActivity，再启动SingleTopActivity，此时不会新建SingleTopActivity。