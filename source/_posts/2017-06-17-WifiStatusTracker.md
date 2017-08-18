---
title: WifiStatusTracker 类
tags:
  - android
  - wifi
date: 2017-06-17 15:37:22
categories: 笔记
---

Android 7.1.1

> 源码位置：platform_frameworks_base/packages/SettingsLib/src/com/android/settingslib/wifi/

​	WifiStatusTracker，**根据广播来追踪 wifi 的状态**，原生设置中使用该这些对象来更新 summary。WifiStatusTracker 提供了handleBroadcast(Intent) 方法来处理广播，相关的广播有：

`WifiManager.WIFI_STATE_CHANGED_ACTION` wifi 状态

`WifiManager.NETWORK_STATE_CHANGED_ACTION` 网络状态

`WifiManager.RSSI_CHANGED_ACTION` 信号强度变化

​	它的构造函数传入了一个 WifiManager 对象

`public WifiStatusTracker(WifiManager wifiManager)`

​	通过 WifiStatusTracker 可以获得以下状态：

+ enabled：wifi是否启动

+ connected：是否连接到 wifi 网络

+ ssid

+ rssi

+ level

  这几个状态都被定义为 public 的变量。