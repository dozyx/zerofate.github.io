---
title: PreferenceActivity 类
tags:
  - android
  - preference
date: 2017-05-21 15:37:22
categories: 笔记
---

> PreferenceActivity 的很多api已经被标记为deprecated，应该使用 PreferenceFragment 替代。

​	PreferenceActivity 是在Activity中向用户展示层级 preference 的基类，它的父类是ListActivity ，每一个item使用 Header 来表示。一般会将header定义在 xml 文件中，并在 onBuildHeaders(...) 方法中使用 loadHeadersFromResource(...) 方法加载，然后header会关联一个 PreferenceFragment 来实现下一级的设置。

​	PreferenceActivity的默认布局为

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_height="match_parent"
    android:layout_width="match_parent">

    <LinearLayout
        android:orientation="horizontal"
        android:layout_width="match_parent"
        android:layout_height="0px"
        android:layout_weight="1">

        <LinearLayout
            style="?attr/preferenceHeaderPanelStyle"
            android:id="@+id/headers"
            android:orientation="vertical"
            android:layout_width="0px"
            android:layout_height="match_parent"
            android:layout_weight="@integer/preferences_left_pane_weight">

            <ListView android:id="@android:id/list"
                style="?attr/preferenceListStyle"
                android:layout_width="match_parent"
                android:layout_height="0px"
                android:layout_weight="1"
                android:clipToPadding="false"
                android:drawSelectorOnTop="false"
                android:cacheColorHint="@android:color/transparent"
                android:listPreferredItemHeight="48dp"
                android:scrollbarAlwaysDrawVerticalTrack="true" />

            <FrameLayout android:id="@+id/list_footer"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:layout_weight="0" />

        </LinearLayout>

        <LinearLayout
                android:id="@+id/prefs_frame"
                style="?attr/preferencePanelStyle"
                android:layout_width="0px"
                android:layout_height="match_parent"
                android:layout_weight="@integer/preferences_right_pane_weight"
                android:orientation="vertical"
                android:visibility="gone" >

            <!-- Breadcrumb inserted here, in certain screen sizes. In others, it will be an
                empty layout or just padding, and PreferenceActivity will put the breadcrumbs in
                the action bar. -->
            <include layout="@layout/breadcrumbs_in_fragment" />

            <android.preference.PreferenceFrameLayout android:id="@+id/prefs"
                    android:layout_width="match_parent"
                    android:layout_height="0dip"
                    android:layout_weight="1"
                />
        </LinearLayout>
    </LinearLayout>

    <!-- button bar -->
</LinearLayout>

```

​	其中，pref_frame 用于装载 header 中的 fragment，这时会将 headers 布局进行隐藏，也就是说，设置的一级、二级界面实际都是使用的同一个布局文件。

​	正如前面所说，PreferenceActivity 可以理解为是一个ListView，它的事件处理在 onHeaderClick 中，如果 header 带有 fragment 属性，则会启动新的Activity。

​	PreferenceActivity 的 ListView 有一个默认的适配器，为PreferenceActivity的内部类 HeaderAdapter，它的父类是一个ArrayAdapter。HeaderAdapter 使用的默认布局为

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:minHeight="48dp"
    android:background="?android:attr/activatedBackgroundIndicator"
    android:gravity="center_vertical"
    android:paddingEnd="?android:attr/scrollbarSize">

    <ImageView
        android:id="@+id/icon"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginStart="6dip"
        android:layout_marginEnd="6dip"
        android:layout_gravity="center" />

    <RelativeLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginStart="2dip"
        android:layout_marginEnd="6dip"
        android:layout_marginTop="6dip"
        android:layout_marginBottom="6dip"
        android:layout_weight="1">

        <TextView android:id="@+android:id/title"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:singleLine="true"
            android:textAppearance="?android:attr/textAppearanceMedium"
            android:ellipsize="marquee"
            android:fadingEdge="horizontal" />

        <TextView android:id="@+android:id/summary"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_below="@android:id/title"
            android:layout_alignStart="@android:id/title"
            android:textAppearance="?android:attr/textAppearanceSmall"
            android:ellipsize="end"
            android:maxLines="2" />

    </RelativeLayout>

</LinearLayout>

```

​	默认的布局比较简单，就一个icon，加上标题和概述。如果需要使用更多样化的布局，则需要自定义自己的HeaderAdapter，并重写 getItemViewType 方法。

### Header

​	PreferenceActivity 将每一项使用一个 Header 来表示，可以在xml文件中使用\<header /\> 标签来表示header，然后在 PreferenceActivity 中加载。\<header \>可以使用的属性有：

```xml
    <!-- Attribute for a header describing the item shown in the top-level list
         from which the selects the set of preference to dig in to. -->
    <declare-styleable name="PreferenceHeader">
        <!-- Identifier value for the header. -->
        <attr name="id" />
        <!-- The title of the item that is shown to the user. -->
        <attr name="title" />
        <!-- The summary for the item. -->
        <attr name="summary" format="string" />
        <!-- The title for the bread crumb of this item. -->
        <attr name="breadCrumbTitle" format="string" />
        <!-- The short title for the bread crumb of this item. -->
        <attr name="breadCrumbShortTitle" format="string" />
        <!-- An icon for the item. -->
        <attr name="icon" />
        <!-- The fragment that is displayed when the user selects this item. -->
        <attr name="fragment" format="string" />
    </declare-styleable>
```