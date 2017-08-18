---
title: Immersive 模式
tags:
  - android
date: 2017-03-19 15:37:22
categories: 笔记
---

[DevBytes: Android 4.4 Immersive Mode](http://www.youtube.com/embed/cBi8fjv90E4)

[Using Immersive Full-Screen Mode](https://developer.android.com/training/system-ui/immersive.html)

​	Immersive full-screen mode，沉浸式全屏模式，个人理解就是一种全屏切换过程中尽量不打断用户注意力的更为自然的模式。该模式随着 `SYSTEM_UI_FLAG_IMMERSIVE` 标记从 API 19开始引入，与这个 flag 类似的还有一个为 `SYSTEM_UI_FLAG_IMMERSIVE_STICKY`。先看下文档注释的说明

+ SYSTEM_UI_FLAG_IMMERSIVE

  ​	用于 setSystemUiVisibility(int) 方法，当使用 `SYSTEM_UI_FLAG_HIDE_NAVIGATION` 隐藏导航栏时，View 将会保持交互。如果没有设置该 flag，用户的任意交互都会导致系统将`SYSTEM_UI_FLAG_HIDE_NAVIGATION`标记强制清除，即恢复为显示状态。

  ​	由于该 flag 是 `SYSTEM_UI_FLAG_HIDE_NAVIGATION` 的一个修饰，所以它只能在与此 flag 组合使用时才能生效。（实际使用时，只与`SYSTEM_UI_FLAG_FULLSCREEN`组合也有效果，不过只是作用于 status bar） 

+ SYSTEM_UI_FLAG_IMMERSIVE_STICKY

  ​	用于 setSystemUiVisibility(int) 方法，当使用 `SYSTEM_UI_FLAG_FULLSCREEN` 隐藏状态栏，和（或者）使用 `SYSTEM_UI_FLAG_HIDE_NAVIGATION` 隐藏导航栏时，View 将会保持交互。如果没有设置该 flag，用户的任意交互都会导致系统将`SYSTEM_UI_FLAG_HIDE_NAVIGATION`标记强制清除，`SYSTEM_UI_FLAG_FULLSCREEN` 标记也会在用户从屏幕顶部往下划动时强制清除。

  ​	当 system bar 在 immersive 模式下隐藏时，可以使用系统手势来将它们短暂显示，如屏幕顶端下划。这些短暂状态的 system bar 将会覆盖在应用内容之上，还会带有一定的透明度，在短时间超时后将自动隐藏。

  ​	**由于该 flag 是 SYSTEM_UI_FLAG_FULLSCREEN 和 SYSTEM_UI_FLAG_HIDE_NAVIGATION 的一个修饰，所以只有与它们组合起来才会有效果。**



​	当处于 imemersive 全屏时（使用 `SYSTEM_UI_FLAG_IMMERSIVE`），activity 将接受所有的触摸事件，用户可以通过从原 system bar 区域向内划动来显示 system bar，这时 `SYSTEM_UI_FLAG_HIDE_NAVIGATION` 和 `SYSTEM_UI_FLAG_FULLSCREEN` 都会被清除，system bar 重新变得可见。同时，还会触发 View.OnSystemUiVisibilityChangeListener 的回调（通过该监听器，我们可以在 system bar 隐藏时，对一些控件进行隐藏）。假如，希望 system bar 可以再次自动隐藏，则应该使用 SYSTEM_UI_FLAG_IMMERSIVE_STICKY ，但 sticky 的方式将不会触发任何的监听器。

​	下图表示处于各个状态时的界面：Non-immersive mode；Reminder bubble；Immersive mode；Sticky flag

![imm-states](https://ws1.sinaimg.cn/large/006tKfTcgy1finvr36128j30je09mmxt.jpg)

####  SYSTEM_UI_FLAG_IMMERSIVE 和 SYSTEM_UI_FLAG_IMMERSIVE_STICKY

两种方式都可以提供 immersive 体验，它们的使用场景将根据的行为体验来选择，如

immersive

+  图书阅读器、新闻阅读、杂志

immersive_sticky

+ 游戏或绘图应用

如果只是需要最小用户交互的视频播放器或者其他应用，则只需要使用 `SYSTEM_UI_FLAG_FULLSCREEN` 和 `SYSTEM_UI_FLAG_HIDE_NAVIGATION`就足够了（注意：点击原 system bar 将会使其重新可见，所以需要在需要时再次主动隐藏），而不需要使用 “immersive” 标记。



#### 使用Non-Sticky Immersion

​	使用 `SYSTEM_UI_FLAG_IMMERSIVE` 时，将会根据设置的 flag 来对相应的 system bar 进行隐藏，但用户从原区域向内划，将会使 system bar 重新可见，并且保持下去。

​	在使用时，最好配合其他 flag，如`SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION` 和 `SYSTEM_UI_FLAG_LAYOUT_STABLE`，来避免在 system bar 显示/隐藏过程中内容大小发生变化。

```java
// This snippet hides the system bars.
private void hideSystemUI() {
    // Set the IMMERSIVE flag.
    // Set the content to appear under the system bars so that the content
    // doesn't resize when the system bars hide and show.
    mDecorView.setSystemUiVisibility(
            View.SYSTEM_UI_FLAG_LAYOUT_STABLE
            | View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
            | View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN
            | View.SYSTEM_UI_FLAG_HIDE_NAVIGATION // hide nav bar
            | View.SYSTEM_UI_FLAG_FULLSCREEN // hide status bar
            | View.SYSTEM_UI_FLAG_IMMERSIVE);
}

// This snippet shows the system bars. It does this by removing all the flags
// except for the ones that make the content appear under the system bars.
private void showSystemUI() {
    mDecorView.setSystemUiVisibility(
            View.SYSTEM_UI_FLAG_LAYOUT_STABLE
            | View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
            | View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN);
}
```

​	还可以使用以下方式来配合 IMMERSIVE 标记，以提供更好的体验：

+ 注册系统 UI 可见性的监听
+ 实现 onWindowFocusChanged()，在获得焦点时隐藏，在失去焦点（如对话框或者弹出的菜单）时显示。
+ 实现 `GestureDetector`来监测 `onSingleTapUp(MotionEvent)`

这些用法，可以在[DevBytes: Android 4.4 Immersive Mode](http://www.youtube.com/embed/cBi8fjv90E4)视频中看到



#### Sticky Immersion

​	使用`SYSTEM_UI_FLAG_IMMERSIVE_STICKY` 时，从 system bar 区域内划将会导致 system bar 暂时显示，但不会有标记被清除，而且 OnSystemUiVisibilityChangeListener 也不会触发，在短暂延时或者用户在屏幕中间进行交互时，system bar 会再次隐藏。注意：在 window 失去焦点时，如弹出对话框，system bar也会再次变得可见，重新获得焦点后会立即隐藏。

​	示例：

```java
@Override
public void onWindowFocusChanged(boolean hasFocus) {
        super.onWindowFocusChanged(hasFocus);
    if (hasFocus) {
        decorView.setSystemUiVisibility(
                View.SYSTEM_UI_FLAG_LAYOUT_STABLE
                | View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
                | View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN
                | View.SYSTEM_UI_FLAG_HIDE_NAVIGATION
                | View.SYSTEM_UI_FLAG_FULLSCREEN
                | View.SYSTEM_UI_FLAG_IMMERSIVE_STICKY);}
}
```

​	如果只是需要 `IMMERSIVE_STICKY `的自动隐藏功能，也可以考虑使用`IMMERSIVE`配合 `Handler.postDelayed()`来实现。