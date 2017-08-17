---
title: StateListDrawable 的状态
tags:
  - android
date: 2017-01-03 15:37:22
categories: 笔记
---



[StateListDrawable](https://developer.android.com/reference/android/graphics/drawable/StateListDrawable.html)

[State List Guide](https://developer.android.com/guide/topics/resources/drawable-resource.html#StateList)

[dither wiki](https://en.wikipedia.org/wiki/Dither#Digital_photography_and_image_processing)

[What is the difference between the states selected, checked and activated in Android?](http://stackoverflow.com/questions/11504860/what-is-the-difference-between-the-states-selected-checked-and-activated-in-and)

StateListDrawable是定义在xml中的drawable对象。每当状态变化时，`第一个匹配的状态将被使用`。

文件位置

res/drawable/filename.xml

语法

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

android:constantSize

为true时选择所有状态的最大值的作为其大小，默认false

android:dither

位图抖动，默认true，为了得到更好的显示效果。[参考](http://blog.csdn.net/superjunjin/article/details/7670864)



### selected, checked and activated区别

1. activated在Honeycomb中开始添加
2. Activated在现在是每一个View都有的属性，有setActivated()和getActivated()方法
3. Checked要求View实现Checkable接口
4. selection是一个短暂的属性，表示用户当前交互的View；activation是一个长时间的状态，如在一个可单选或多选的list view中，当前选中的view的状态为activated

关于上面第4点，还存在一些疑问，在使用d-pad移动时，selected可能（实际使用中没有得到验证）是可以保持的，而使用setSelection设置时，selected状态无法保持，具体情况需要根据实际使用做出判断，参照官方文档：

+ **android:state_selected**
  Boolean. "true" if this item should be used when the object is the current user selection when navigating with a directional control (such as when navigating through a list with a d-pad); "false" if this item should be used when the object is not selected.
  The selected state is used when focus (android:state_focused) is not sufficient (such as when list view has focus and an item within it is selected with a d-pad).
+ **android:state_checked**
  Boolean. "true" if this item should be used when the object is checked; "false" if it should be used when the object is un-checked.
+ **android:state_activated**
  Boolean. "true" if this item should be used when the object is activated as the persistent selection (such as to "highlight" the previously selected list item in a persistent navigation view); "false" if it should be used when the object is not activated.
  Introduced in API level 11.

















