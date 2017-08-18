---
title: PreferenceFragment 类
tags:
  - android
  - preference
date: 2017-04-19 15:37:22
categories: 笔记
---

​	PreferenceFragment 继承于 Fragment，它将 Preference 显示为 List 的形式。

​	在 onCreateView 中默认的style为

```xml
    <style name="PreferenceFragment">
        <item name="layout">@layout/preference_list_fragment</item>
        <item name="paddingStart">0dp</item>
        <item name="paddingEnd">0dp</item>
    </style>
```

​	preference_list_fragment.xml

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_height="match_parent"
    android:layout_width="match_parent"
    android:background="@android:color/transparent"
    android:layout_removeBorders="true">

    <ListView android:id="@android:id/list"
        style="?attr/preferenceFragmentListStyle"
        android:layout_width="match_parent"
        android:layout_height="0px"
        android:layout_weight="1"
        android:paddingTop="0dip"
        android:paddingBottom="@dimen/preference_fragment_padding_bottom"
        android:scrollbarStyle="@integer/preference_fragment_scrollbarStyle"
        android:clipToPadding="false"
        android:drawSelectorOnTop="false"
        android:cacheColorHint="@android:color/transparent"
        android:scrollbarAlwaysDrawVerticalTrack="true" />

    <TextView android:id="@android:id/empty"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:padding="@dimen/preference_fragment_padding_side"
        android:gravity="center"
        android:visibility="gone" />

    <!-- button bar -->
</LinearLayout>
```