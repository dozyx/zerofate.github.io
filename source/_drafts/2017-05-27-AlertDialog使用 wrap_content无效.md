---
title: AlertDialog 使用 wrap_content 无效
tags:
  - android
  - 对话框
date: 2017-05-27 15:37:22
categories: 笔记
---

使用 AlertDialog 时，设置custom view 的layout _width 为 wrap_content 无作用，导致两边有较大空白

原因：应用的主题中设置了alertDialogTheme，它引用的的 style 定义了对话框window的最小宽度，如 AppCompat 主题默认就使用了以下style

```xml
<style name="Base.ThemeOverlay.AppCompat.Dialog.Alert">
   <item name="windowMinWidthMajor">@dimen/abc_dialog_min_width_major</item>
   <item name="windowMinWidthMinor">@dimen/abc_dialog_min_width_minor</item>
</style>
```

major用于屏幕主轴，minor用于屏幕次轴。major对应的大小为65%，minor为95%。

如果需要修改，可以自定义一个style，然后将最小宽度设为0

```xml
<style name="DialogStyle" >
   <item name="android:windowBackground">@android:color/transparent</item>
   <item name="android:windowFrame">@null</item>
   <item name="android:windowIsFloating">true</item>
   <item name="windowMinWidthMajor">0%</item>
</style>
```