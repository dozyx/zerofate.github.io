---
title: Preference 相关布局修改
tags:
  - android
  - preference
date: 2017-05-19 15:37:22
categories: 笔记
---

## PreferenceFragment 布局

​	PreferenceFragment 的自定义属性在 `com.android.internal.R.styleable.PreferenceFragment`中，其中定义了`layout`和`divider`两个属性（可能不同 API 版本有差别）。同时，PreferenceFragment 还有一个主题引用属性`com.android.internal.R.attr.preferenceFragmentStyle`。

​	默认的PreferenceFragment 布局在 preference_list_fragment.xml 中。

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

    <!-- 底部还有个可见性为 gone 的button bar -->
  	...
</LinearLayout>

```

 	可以通过向 theme 添加preferenceFragmentStyle 属性，并指向指定了`android:layout`属性的style来修改布局。这种方式会对每个 PreferenceFragment 生效。比如原生设置（Android 7.1.1）的自定义布局为

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/container_material"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@android:color/transparent">

    <FrameLayout android:id="@+id/pinned_header"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:visibility="gone" />

    <FrameLayout
        android:id="@android:id/list_container"
        android:layout_height="0px"
        android:layout_weight="1"
        android:layout_width="match_parent"
        android:paddingStart="@dimen/settings_side_margin"
        android:paddingEnd="@dimen/settings_side_margin">

        <ListView android:id="@+id/backup_list"
            style="@style/PreferenceFragmentListSinglePane"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:paddingTop="@dimen/dashboard_padding_top"
            android:paddingBottom="@dimen/dashboard_padding_bottom"
            android:scrollbarStyle="@*android:integer/preference_fragment_scrollbarStyle"
            android:clipToPadding="false"
            android:drawSelectorOnTop="false"
            android:elevation="@dimen/dashboard_category_elevation"
            android:visibility="gone"
            android:scrollbarAlwaysDrawVerticalTrack="true" />

        <include layout="@layout/loading_container" />

        <com.android.settings.widget.FloatingActionButton
            android:id="@+id/fab"
            android:visibility="gone"
            android:clickable="true"
            android:layout_width="@dimen/fab_size"
            android:layout_height="@dimen/fab_size"
            android:layout_gravity="bottom|end"
            android:layout_marginEnd="@dimen/fab_margin"
            android:layout_marginBottom="@dimen/fab_margin"
            android:elevation="@dimen/fab_elevation"
            android:background="@drawable/fab_background" />

    </FrameLayout>

    <TextView android:id="@android:id/empty"
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        android:padding="@*android:dimen/preference_fragment_padding_side"
        android:layout_gravity="center"
        android:gravity="center_vertical"
        android:visibility="gone" />

    <include layout="@layout/admin_support_details_empty_view" />

	<!-- button bar -->
  	...
</LinearLayout>
```

自定义布局在 ListView 底部设置了一个 header 布局。

​	在一些旧Android版本，无法使用 preferenceFragmentStyle 属性，如 Android 4.4 ，其PreferenceFragment 的onCreateView实现为

```java
@Override 
public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) { 		return inflater.inflate(com.android.internal.R.layout.preference_list_fragment, container, false); 
}
```

​	所以，我们只需要简单的重写 onCreateView 并返回自定义View即可。Android 7.1.1 的实现实际上也只是将视图进行了 inflate 而已。 



## PreferenceScreen 布局

​	PreferenceScreen 间接继承了 Preference，其 style 可以通过在 theme 中设置 preferenceScreenStyle 属性来指定。默认的PreferenceScreen没有自定义其布局，即直接使用的 Preference 的布局。

## PreferenceCategory 布局

​	PreferenceCategory 用于对Preference 进行分组，并在组上面提供一个 disabled 状态的标题。PreferenceCategory也是间接继承了Preference，除了直接在 xml 中设置 android:layout 外还可以 通过 preferenceCategoryStyle 属性定义它的style。

​	默认 style 

```xml
<style name="Preference.Category">
	<item name="layout">@layout/preference_category</item>
<!-- The title should not dim if the category is disabled, instead only the preference children should dim. -->
	<item name="shouldDisableView">false</item>
	<item name="selectable">false</item>
</style>
```

​	preference_category.xml

```xml
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    style="?android:attr/listSeparatorTextViewStyle"
    android:id="@+android:id/title"
/>
```

​	`style="?android:attr/listSeparatorTextViewStyle"`会在 TextView 的底部添加一条分隔线背景。



















