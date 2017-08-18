---
title: AccessPoint 类
tags:
  - android
  - wifi
date: 2017-04-27 15:37:22
categories: 笔记
---

> AccessPoint 类位于[platform_frameworks_base](https://github.com/android/platform_frameworks_base)/[packages](https://github.com/android/platform_frameworks_base/tree/master/packages)/[SettingsLib](https://github.com/android/platform_frameworks_base/tree/master/packages/SettingsLib)/[src](https://github.com/android/platform_frameworks_base/tree/master/packages/SettingsLib/src)/[com](https://github.com/android/platform_frameworks_base/tree/master/packages/SettingsLib/src/com)/[android](https://github.com/android/platform_frameworks_base/tree/master/packages/SettingsLib/src/com/android)/[settingslib](https://github.com/android/platform_frameworks_base/tree/master/packages/SettingsLib/src/com/android/settingslib)/[wifi](https://github.com/android/platform_frameworks_base/tree/master/packages/SettingsLib/src/com/android/settingslib/wifi)/



​	AccessPoint 类用来表示一个接入点。



### Comparable比较器

​	AccessPoint 实现了 Comparable 接口以进行排序。其 compareTo 方法按以下规则对每个接入点进行排序：

+ 接入点是否有效

+ 没有信号强度的放在最后（单位为dBm，为负值，绝对值越大，信号越强）（个人觉得应该是已连接过，但无信号的接入点）

+ 接入点是否已配置

+ 信号强度

+ 最后按 SSID 名称排序

  ​