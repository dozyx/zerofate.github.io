---
title: StateListDrawable
tags:
  - android
date: 2017-04-28 15:37:22
categories: 笔记
---

[State List](https://developer.android.com/guide/topics/resources/drawable-resource.html#StateList)

​	StateListDrawable 定义在XML中，可以根据对象状态的不同，使用不同的图片来表示同一图像。如，Button 可以根据状态（pressed、focused等）的不同设置不同的背景。

​	**当状态改变时，state list 由上往下寻找第一个匹配的状态，而不是根据最佳匹配原则。**在代码中，通过Drawable.setState(int[] stateSet) 来设置Drawable的状态，Drawable对象会选择 stateSet 中属性均为true的item对应的drawable。



### **语法**

```xml
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android"
    android:constantSize=["true" | "false"]
    android:dither=["true" | "false"]
    android:variablePadding=["true" | "false"] >
    <item
        android:drawable="@[package:]drawable/drawable_resource"
        android:state_pressed=["true" | "false"]
        android:state_focused=["true" | "false"]
        android:state_hovered=["true" | "false"]
        android:state_selected=["true" | "false"]
        android:state_checkable=["true" | "false"]
        android:state_checked=["true" | "false"]
        android:state_enabled=["true" | "false"]
        android:state_activated=["true" | "false"]
        android:state_window_focused=["true" | "false"] />
</selector>
```

### **元素**

*android:constantSize*

为 true 时表示drawable的内置大小在状态变化时不发生变化（大小为所有 state 中的最大值）

*android:dither*

是否允许 bitmap 的抖动

*android:variablePadding*

true  表示drawable 的内边距会基于当前选择的状态发生改变，false则保持一样（基于所有 state 的内边距最大值）



*android:state_hovered*

鼠标悬停状态（不过在模拟器上试验没作用，设备不支持？）



### **示例**

 res/drawable/button.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:state_pressed="true"
          android:drawable="@drawable/button_pressed" /> <!-- pressed -->
    <item android:state_focused="true"
          android:drawable="@drawable/button_focused" /> <!-- focused -->
    <item android:state_hovered="true"
          android:drawable="@drawable/button_focused" /> <!-- hovered -->
    <item android:drawable="@drawable/button_normal" /> <!-- default -->
</selector>
```

对Button应用

```xml
<Button
    android:layout_height="wrap_content"
    android:layout_width="wrap_content"
    android:background="@drawable/button" />
```