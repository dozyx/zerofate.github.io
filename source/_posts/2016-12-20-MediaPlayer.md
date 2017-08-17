---
title: MediaPlayer - release vs reset
tags:
  - android
  - media
date: 2016-12-20 15:37:22
categories: 笔记
---

[release()](https://developer.android.com/reference/android/media/MediaPlayer.html#release())

[reset()](https://developer.android.com/reference/android/media/MediaPlayer.html#reset())

release()将MediaPlayer相关的资源释放，当使用完MediaPlayer后需要调用此方法；reset()将MediaPlayer重设到一个未初始化的状态，调用后需要重新初始化并调用prepare()。从MediaPlayer的生命周期图可以看到这两个方法执行后的流向。

源码分析