---
title: LevelListDrawable
tags:
  - android
date: 2017-04-28 15:37:22
categories: 笔记
---

​	根据指定的最大数值来选择 LevelListDrawable中 的Drawable。在 level list 中，通过 setLevel() 来设置drawable 的层次值，从而加载对应的图片资源。



### 语法

```xml
<?xml version="1.0" encoding="utf-8"?>
<level-list
    xmlns:android="http://schemas.android.com/apk/res/android" >
    <item
        android:drawable="@drawable/drawable_resource"
        android:maxLevel="integer"
        android:minLevel="integer" />
</level-list>
```



### 示例

#### 简单示例

```xml
<?xml version="1.0" encoding="utf-8"?>
<level-list xmlns:android="http://schemas.android.com/apk/res/android" >
    <item
        android:drawable="@drawable/status_off"
        android:maxLevel="0" />
    <item
        android:drawable="@drawable/status_on"
        android:maxLevel="1" />
</level-list>
```



#### wifi 信号drawable

wifi信号所使用的 drawable 定义在`wifi_signal.xml` 文件中

```xml
<selector xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:settings="http://schemas.android.com/apk/res/com.android.settings">
    <item settings:state_encrypted="true" android:drawable="@drawable/wifi_signal_lock_dark" />
    <item settings:state_encrypted="false" android:drawable="@drawable/wifi_signal_open_dark" />
</selector
```

`wifi_signal_lock_dark.xml`

```xml
<level-list xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:maxLevel="0" android:drawable="@drawable/ic_wifi_lock_signal_1_dark" />
    <item android:maxLevel="1" android:drawable="@drawable/ic_wifi_lock_signal_2_dark" />
    <item android:maxLevel="2" android:drawable="@drawable/ic_wifi_lock_signal_3_dark" />
    <item android:maxLevel="3" android:drawable="@drawable/ic_wifi_lock_signal_4_dark" />
</level-list
```



源码分析：

> 源码位置为Android 7.1.1 的platform_frameworks_base/packages/SettingsLib/src/com/android/settingslib/wifi/AccessPointPreference.java 

AccessPointPreference 表示一个接入点在列表中的表现形式。

```java
	private static final int[] STATE_SECURED = {
            R.attr.state_encrypted
    };
    private static final int[] STATE_NONE = {};
	...
      
	private final StateListDrawable mWifiSld;//该变量用于存储表示wifi信号的StateListDrawable
	...
    
    protected void updateIcon(int level, Context context) {
      	//在此方法中对信号图标进行修改
        if (level == -1) {
            safeSetDefaultIcon();
        } else {
            if (getIcon() == null) {
                // To avoid a drawing race condition, we first set the state (SECURE/NONE) and then
                // set the icon (drawable) to that state's drawable.
                // If sld is null then we are indexing and therefore do not have access to
                // (nor need to display) the drawable.
                if (mWifiSld != null) {
                    mWifiSld.setState((mAccessPoint.getSecurity() != AccessPoint.SECURITY_NONE)
                            ? STATE_SECURED
                            : STATE_NONE);//根据state获取对应的drawable，STATE_SECURED选择state_encrypted
                  						  //为true的drawable
                    Drawable drawable = mWifiSld.getCurrent();//获取当前的drawable（wifi中为加密和未加密）
                    if (!mForSavedNetworks && drawable != null) {
                        setIcon(drawable);
                      	//setIcon将调用内部的notifyChanged，最终导致onBindView的回调（v7支持库为
                        //onBindViewHolder）,从而根据level设置drawable
                        return;
                    }
                }
                safeSetDefaultIcon();
            }
        }
    }
    private void safeSetDefaultIcon() {
        if (mDefaultIconResId != 0) {
            setIcon(mDefaultIconResId);
        } else {
            setIcon(null);
        }
    }

    @Override
    public void onBindViewHolder(final PreferenceViewHolder view) {
        super.onBindViewHolder(view);
        if (mAccessPoint == null) {
            // Used for dummy pref.
            return;
        }
        Drawable drawable = getIcon();
        if (drawable != null) {
            drawable.setLevel(mLevel);//根据level选择drawable
        }

        mTitleView = (TextView) view.findViewById(com.android.internal.R.id.title);
        if (mTitleView != null) {
            // Attach to the end of the title view
            mTitleView.setCompoundDrawablesRelativeWithIntrinsicBounds(null, null, mBadge, null);
            mTitleView.setCompoundDrawablePadding(mBadgePadding);
        }
        view.itemView.setContentDescription(mContentDescription);
    }
```


