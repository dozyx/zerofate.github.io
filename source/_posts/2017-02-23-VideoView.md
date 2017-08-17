---
title: VideoView
tags:
  - android
  - media
date: 2017-02-23 15:37:22
categories: 笔记
---

​	用于显示视频文件。VideoView 的父类是 SurfaceView ，SurfaceView提供了一个内嵌在 View 层级中可以进行专门绘制的surface，SurfaceView的父类为View。

> 注意：当进入后台时，VideoView 无法维持它的完全状态（full state）。特别是，它不能恢复当前播放状态、播放位置、选择的曲目或者是通过	`addSubtitleSource()` 添加的字幕。应用需要在`Activity.onSaveInstanceState` 和 `Activity.onRestoreInstaceState` 中自行保存和恢复。同样的，音频的 session id 也可能在恢复时发生改变。

​	VideoView 和 SurfaceView 都没有自定义的属性，所以在Xml中使用的是继承于 View 类的属性。

## VideoView播放的所有状态

在不同的 Android 可能会有差别，这些状态主要与MediaPlayer 的状态图相关

```java
    private static final int STATE_ERROR              = -1;
    private static final int STATE_IDLE               = 0;
    private static final int STATE_PREPARING          = 1;
    private static final int STATE_PREPARED           = 2;
    private static final int STATE_PLAYING            = 3;
    private static final int STATE_PAUSED             = 4;
    private static final int STATE_PLAYBACK_COMPLETED = 5;
```


