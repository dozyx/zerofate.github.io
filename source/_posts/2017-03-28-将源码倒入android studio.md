---
title: 将 Android 源码导入 AS
tags:
  - android
date: 2017-03-28 15:37:22
categories: 笔记
---

Mac系统：

+ 打开终端，在AOSP目录下执行

  `make idegen && development/tools/idegen/idegen.sh`

+ 上一步执行完成后将在AOSP目录生成一个 android.ipr 文件，在android studio 中 open project ，选择该文件打开

+ 等待完成

