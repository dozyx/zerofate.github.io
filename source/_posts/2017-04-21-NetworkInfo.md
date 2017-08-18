---
title: NetworkInfo 类
tags:
  - android
  - wifi 
date: 2017-04-21 15:37:22
categories: 笔记
---

NetworkInfo 类用于**描述网络接口的状态**，可通过 ConnectivityManager 的 getActiveNetworkInfo() 来获得表示当前网络连接的一个实例。

### NetworkInfo.State 和 NetworkInfo.DetailedState

State 表示粗粒度的网络状态，而DetailedState是细粒度的网络状态。大部分应用使用的是 **State** 而不是 **DetailedState**，它们的对应关系如下：

| **Detailed state** | **Coarse-grained state** |
| ------------------ | ------------------------ |
| `IDLE`             | `DISCONNECTED`           |
| `SCANNING`         | `CONNECTING`             |
| `CONNECTING`       | `CONNECTING`             |
| `AUTHENTICATING`   | `CONNECTING`             |
| `CONNECTED`        | `CONNECTED`              |
| `DISCONNECTING`    | `DISCONNECTING`          |
| `DISCONNECTED`     | `DISCONNECTED`           |
| `UNAVAILABLE`      | `DISCONNECTED`           |
| `FAILED`           | `DISCONNECTED`           |

```java
public enum State {
        CONNECTING, CONNECTED, SUSPENDED, DISCONNECTING, DISCONNECTED, UNKNOWN
    }
public enum DetailedState {
        /** Ready to start data connection setup. */
        IDLE,
        /** Searching for an available access point. */
        SCANNING,
        /** Currently setting up data connection. */
        CONNECTING,
        /** Network link established, performing authentication. */
        AUTHENTICATING,
        /** Awaiting response from DHCP server in order to assign IP address information. */
        OBTAINING_IPADDR,
        /** IP traffic should be available. */
        CONNECTED,
        /** IP traffic is suspended */
        SUSPENDED,
        /** Currently tearing down data connection. */
        DISCONNECTING,
        /** IP traffic not available. */
        DISCONNECTED,
        /** Attempt to connect failed. */
        FAILED,
        /** Access to this network is blocked. */
        BLOCKED,
        /** Link has poor connectivity. */
        VERIFYING_POOR_LINK,
        /** Checking if network is a captive portal */
        CAPTIVE_PORTAL_CHECK
    }

```



### 部分方法

`public boolean isConnected()`

### info类型

`public int getType()`

可能的类型有：

ConnectivityManager.TYPE_MOBILE

ConnectivityManager.TYPE_WIFI

ConnectivityManager.TYPE_WIMAX

ConnectivityManager.TYPE_ETHERNET

ConnectivityManager.TYPE_BLUETOOTH



`public String getTypeName()`

返回具有可读性的名称，如“WIFI“、”MOBILE“



### getReason

建立连接失败的原因，null表示此方法不可用。