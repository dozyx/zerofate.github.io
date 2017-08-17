---
title: 读取 Activity 的 label
tags:
  - android
date: 2017-08-14 15:37:22
categories: 笔记
---

两种方式：

+ ActivityInfo

  首先通过 PackageManager 获取到特定 Activity （Intent 或 component 名）的 ActivityInfo 信息，然后使用 ActivityInfo 的 labelRes 获取 label 的资源 id。**如果没有设置 label 属性，获取到的 labelRes 为0。**

+ ResolveInfo

  使用 ResolveInfo 的 loadLabel(PackageManager) 方法。




**ActivityInfo 同样有一个 loadLabel 方法，该方法实现在其父类 ComponentInfo 中，暂时没发现与 ResolveInfo 的区别**