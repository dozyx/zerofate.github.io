---
title: WifiManager 类
tags:
  - android
  - wifi 
date: 2017-06-16 15:37:22
categories: 笔记
---

### 连接

​	WifiManager 提供了两个 connect 方法，一个需要用于连接新网络，一个用于连接已配置网络。在连接新网络时，该网络将添加进客户端的配置中。

`public void connect(WifiConfiguration config, ActionListener listener) `

`public void connect(int networkId, ActionListener listener)`

​	对于新的网络，connect(WifiConfiguration,ActionListener) 相当于依次执行 addNetwork()、enableNetwork()、saveConfiguration()、reconnect()；connect(int, ActionListener) 相当于执行enableNetwork()、saveConfiguration()、reconnect()。

> addNetwork() 会将网络添加到已配置网络中，而 saveConfiguration() 将对已配置网络列表中的所有网络进行持久化保存。saveConfiguration() 可能导致网络的 network Id 发生改变。



### 取消保存

​	从客户端配置中删除该网络。相当于依次执行 removeNetwork()、saveConfiguration()。

` public void forget(int netId, ActionListener listener)`



### 保存

​	将指定网络保存到客户端配置中，如果该网络已存在，其配置将被更新，一个新的网络将默认被启用。对于新的网络，该方法相当于addNetwork()、enableNetwork()、saveConfiguration()，对于已存在的网络，相当于执行updateNetwork()、saveConfiguration()。

`public void save(WifiConfiguration config, ActionListener listener)`



### addnetwork()

将一个新的网络添加到已配置网络的集合中，config 对象的 networkId 将被忽略。新网络将默认被标记为 DISABLED。

`public int addNetwork(WifiConfiguration config)`

### enabledNetwork()

允许连接先前配置的网络

`public boolean enableNetwork(int netId, boolean disableOthers)`

### saveConfiguration()

通知客户端对已配置网络列表进行持久化保存，该方法可能导致已存在网络的 network Id 发生改变。

`public boolean saveConfiguration()`

**注：这里并不是对单个网络配置进行保存，而是将所有的配置网络保存到文件中。**

### reconnect()

如果最近断开了连接，则对最近活动的接入点进行重新连接。

`public boolean reconnect()`