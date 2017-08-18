---
title: StorageVolume 类
tags:
  - android
date: 2017-03-20 15:37:22
categories: 笔记
---

​	为特定用户提供了共享/外部存储器（storage volume）的信息。该类从API 24开始公开。

​	一个设备可以有一个（只能有一个）主要（primary）存储器，但仍然可以有额外的存储器，如SD卡和U盘。StorageVolume对象代表特定用户（不同用户看到的同一物理卷（physical volume）可能不一样，如内置的模拟存储）使用的存储器的逻辑视图。

​	读写存储器需要先请求权限，可以通过以下方式获取：

+ 推荐方式：createAccessIntent(String) 请求访问目录
+ 使用framework的存储访问api，如 [ACTION_OPEN_DOCUMENT](https://developer.android.com/reference/android/content/Intent.html#ACTION_OPEN_DOCUMENT) 和[ACTION_OPEN_DOCUMENT_TREE](https://developer.android.com/reference/android/content/Intent.html#ACTION_OPEN_DOCUMENT_TREE)。（Intent方式）
+ 可以声明 `READ_EXTERNAL_STORAGE` 和 `WRITE_EXTERNAL_STORAGE`来读写主要存储器



### method

#### isPrimary()

​	判断该volume是否为主要的共享/外部存储器，主要存储器可通过 `Environment.getExternalStorageDirectory()`获得。