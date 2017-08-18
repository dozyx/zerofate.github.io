---
title: Preference 的一些子类
tags:
  - android
  - preference
date: 2017-05-23 15:37:22
categories: 笔记
---

## SeekBarPreference

​	SeekBarPreference 定义在 v7 支持库中，基于 v7 的Preference（其实，在sdk中也有一个，但被标记为 @hide）。除了直接使用库里的SeekBarPreference，我们也可以直接拷贝源码自己修改。下面内容基于 v7 库中的SeekBarPreference。

### 自定义属性

​	SeekBarPreference 的自定义属性包括：

+ min
+ max
+ seekBarIncrement：integer，每次方向键按下时的增量值
+ adjustable ：boolean，如果为true 将响应 left/right 物理键
+ showSeekBarValue

### 布局

​	SeekBarPreference 包括了一个标题、一个seekbar，还有一个可选的用于显示seekbar值得TextView。它的布局可以通过 android:layout 里的 widget frame 或者利用 seekbarPreferenceStyle 属性来自定义。

​	默认布局在 preference_widget_seekbar.xml 文件中，seekbar和显示值的TextView放置在了summary下面

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="wrap_content"
              android:minHeight="?android:attr/listPreferredItemHeight"
              android:gravity="center_vertical"
              android:paddingEnd="?android:attr/scrollbarSize"
              android:clipChildren="false"
              android:clipToPadding="false">
    <ImageView
            android:id="@+android:id/icon"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:minWidth="@dimen/preference_icon_minWidth"/>
    <RelativeLayout
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginStart="16dip"
            android:layout_marginEnd="8dip"
            android:layout_marginTop="6dip"
            android:layout_marginBottom="6dip"
            android:layout_weight="1"
            android:clipChildren="false"
            android:clipToPadding="false">
        <TextView android:id="@+android:id/title"
                  android:layout_width="wrap_content"
                  android:layout_height="wrap_content"
                  android:singleLine="true"
                  android:textAppearance="?android:attr/textAppearanceMedium"
                  android:ellipsize="marquee"
                  android:fadingEdge="horizontal"/>
        <TextView android:id="@+android:id/summary"
                  android:layout_width="wrap_content"
                  android:layout_height="wrap_content"
                  android:layout_below="@android:id/title"
                  android:layout_alignStart="@android:id/title"
                  android:textAppearance="?android:attr/textAppearanceSmall"
                  android:textColor="?android:attr/textColorSecondary"
                  android:maxLines="4"/>
        <!-- Using UnPressableLinearLayout as a workaround to disable the pressed state propagation
        to the children of this container layout. Otherwise, the animated pressed state will also
        play for the thumb in the AbsSeekBar in addition to the preference's ripple background.
        The background of the SeekBar is also set to null to disable the ripple background -->
        <android.support.v7.preference.UnPressableLinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_below="@android:id/summary"
                android:layout_alignStart="@android:id/title"
                android:clipChildren="false"
                android:clipToPadding="false">
            <SeekBar
                    android:id="@+id/seekbar"
                    android:layout_width="0dp"
                    android:layout_weight="1"
                    android:layout_height="wrap_content"
                    android:paddingStart="@dimen/preference_seekbar_padding_start"
                    android:paddingEnd="@dimen/preference_seekbar_padding_end"
                    android:focusable="false"
                    android:clickable="false"
                    android:background="@null" />
            <TextView android:id="@+id/seekbar_value"
                      android:layout_width="@dimen/preference_seekbar_value_width"
                      android:layout_height="match_parent"
                      android:gravity="right|center_vertical"
                      android:fontFamily="sans-serif-condensed"
                      android:singleLine="true"
                      android:textAppearance="?android:attr/textAppearanceMedium"
                      android:ellipsize="marquee"
                      android:fadingEdge="horizontal"/>
        </android.support.v7.preference.UnPressableLinearLayout>
    </RelativeLayout>
</LinearLayout>
```

​	了解上面的布局并不就表示 SeekBarPreference 就应该都是这样的，应该以一种更为变通的方式来思考。在这个布局里其实对于 SeekBarPreference 来说，只有一个 seekbar 是关键，我们可以随意调整布局，只要确保 id 正确。如果需要更完全的自定义，还可以完全继承Preference来实现自己的SeekBarPreference，比如在原生设置（Android 7.1.1 里的音量调节，就是自定义了SeekBarPreference）。







