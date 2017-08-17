---
title: 使 RadioButton 文本显示在右侧
tags:
  - android
date: 2017-05-31 15:37:22
categories: 笔记
---

[android RadioButton option on the right side of the text](https://stackoverflow.com/questions/1886681/android-radiobutton-option-on-the-right-side-of-the-text)

使用CheckTextView

```xml
<?xml version="1.0" encoding="utf-8"?>
<CheckedTextView 
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@android:id/text1"
    android:layout_width="match_parent"
    android:layout_height="?android:attr/listPreferredItemHeightSmall"
    android:textAppearance="?android:attr/textAppearanceListItemSmall"
    android:gravity="center_vertical"
    android:checkMark="?android:attr/listChoiceIndicatorSingle"
    android:paddingStart="?android:attr/listPreferredItemPaddingStart"
    android:paddingEnd="?android:attr/listPreferredItemPaddingEnd" />
```

