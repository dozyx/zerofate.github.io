---
title: AlertDialog
tags:
  - android
  - 对话框
date: 2017-01-04 15:37:22
categories: 笔记
---

参考

[How to style AlertDialogs like a pro](http://blog.supenta.com/2014/07/02/how-to-style-alertdialogs-like-a-pro/)

AlertDialog通常有三个部分：title、message、button；可以在theme中指定与AlertDialog相关的两个属性：alertDialogTheme和alertDialogStyle，预置的AlertDialog主题有：

+ THEME_DEVICE_DEFAULT_DARK
+ THEME_DEVICE_DEFAULT_LIGHT
+ THEME_HOLO_DARK
+ THEME_HOLO_LIGHT
+ THEME_TRADITIONAL

![dialogs_themes](/Users/zero/OneDrive/markdown\photo\dialog\dialogs_themes.png)

AlertDialog的主题属性

```xml
<!-- The set of attributes that describe a AlertDialog's theme. -->
	<declare-styleable name="AlertDialog">
		<attr name="fullDark" format="reference|color" />
      
        <!-- topDark：标题背景 -->
        <attr name="topDark" format="reference|color" />
      
      	<!-- centerDark：message背景 -->
        <attr name="centerDark" format="reference|color" />
      
      	<!-- bottomDark：button bar背景 -->
        <attr name="bottomDark" format="reference|color" />
      
        <attr name="fullBright" format="reference|color" />
        <attr name="topBright" format="reference|color" />
        <attr name="centerBright" format="reference|color" />
        <attr name="bottomBright" format="reference|color" />
        <attr name="bottomMedium" format="reference|color" />
        <attr name="centerMedium" format="reference|color" />
        <attr name="layout" />
        <attr name="buttonPanelSideLayout" format="reference" />
        <attr name="listLayout" format="reference" />
        <attr name="multiChoiceItemLayout" format="reference" />
        <attr name="singleChoiceItemLayout" format="reference" />
        <attr name="listItemLayout" format="reference" />
        <attr name="progressLayout" format="reference" />
        <attr name="horizontalProgressLayout" format="reference" />
        <!-- @hide Whether fullDark, etc. should use default values if null. -->
        <attr name="needsDefaultBackgrounds" format="boolean" />
    </declare-styleable>
```