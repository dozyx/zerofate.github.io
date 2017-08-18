---
title: WifiTracker 类
tags:
  - android
  - wifi
date: 2017-04-28 15:37:22
categories: 笔记
---

Android 7.1.1

> 源码位置：platform_frameworks_base/packages/SettingsLib/src/com/android/settingslib/wifi

​	WifiTracker 用来跟踪已保存或可用的 wifi 网络，以及它们的状态。与 WifiStatusTracker 比较，WifiStatusTracker 的作用在于跟踪 wifi 本身的状态（是否开启、是否连接、ssid等）。WifiTracker 具有多个构造函数，但最终调用的是同一个，但该构造函数被标记为 @VisibleForTesting，下面的公开可见的最后一个构造函数

```java
public WifiTracker(Context context, WifiListener wifiListener, Looper workerLooper,
            boolean includeSaved, boolean includeScans, boolean includePasspoints)
```

### wifi 网络跟踪

​	WifiTacker 对以下广播进行了监听：

`WifiManager.WIFI_STATE_CHANGED_ACTION`：wifi 状态

`WifiManager.SCAN_RESULTS_AVAILABLE_ACTION`：扫描结果可用

`WifiManager.NETWORK_IDS_CHANGED_ACTION`

`WifiManager.SUPPLICANT_STATE_CHANGED_ACTION`

`WifiManager.CONFIGURED_NETWORKS_CHANGED_ACTION`

`WifiManager.LINK_CONFIGURATION_CHANGED_ACTION`

`WifiManager.NETWORK_STATE_CHANGED_ACTION`

​	startTracking() 将这些广播进行了注册监听（同时也会启动网络扫描），而stopTracking() 将解除监听。

#### 广播处理

​	广播的处理分成了三类：

+ wifi 状态：指 wifi 本身这个功能
+ wifi 网络
+ wifi 连接状态

*WifiTracker*

```java
    @VisibleForTesting
    final BroadcastReceiver mReceiver = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            String action = intent.getAction();
            if (WifiManager.WIFI_STATE_CHANGED_ACTION.equals(action)) {
                updateWifiState(intent.getIntExtra(WifiManager.EXTRA_WIFI_STATE,
                        WifiManager.WIFI_STATE_UNKNOWN));//开始/停止扫描并回调监听器
            } else if (WifiManager.SCAN_RESULTS_AVAILABLE_ACTION.equals(action) ||
                    WifiManager.CONFIGURED_NETWORKS_CHANGED_ACTION.equals(action) ||
                    WifiManager.LINK_CONFIGURATION_CHANGED_ACTION.equals(action)) {
                mWorkHandler.sendEmptyMessage(WorkHandler.MSG_UPDATE_ACCESS_POINTS);//更新接入点信息
            } else if (WifiManager.NETWORK_STATE_CHANGED_ACTION.equals(action)) {
                NetworkInfo info = (NetworkInfo) intent.getParcelableExtra(
                        WifiManager.EXTRA_NETWORK_INFO);
                mConnected.set(info.isConnected());

                mMainHandler.sendEmptyMessage(MainHandler.MSG_CONNECTED_CHANGED);//回调监听器

                mWorkHandler.sendEmptyMessage(WorkHandler.MSG_UPDATE_ACCESS_POINTS);
                mWorkHandler.obtainMessage(WorkHandler.MSG_UPDATE_NETWORK_INFO, info)
                        .sendToTarget();
            }
        }
    };
```

#### updateAccessPoints

​	更新接入点信息

```java
private void updateAccessPoints() {
        // Swap the current access points into a cached list.
        List<AccessPoint> cachedAccessPoints = getAccessPoints();//缓存旧的接入点信息
        ArrayList<AccessPoint> accessPoints = new ArrayList<>();

        // Clear out the configs so we don't think something is saved when it isn't.
        for (AccessPoint accessPoint : cachedAccessPoints) {
            accessPoint.clearConfig();//清除 AccessPoint 的配置信息
        }

        /** Lookup table to more quickly update AccessPoints by only considering objects with the
         * correct SSID.  Maps SSID -> List of AccessPoints with the given SSID.  */
        Multimap<String, AccessPoint> apMap = new Multimap<String, AccessPoint>();
        WifiConfiguration connectionConfig = null;
        if (mLastInfo != null) {
          	//获取当前连接对应的配置
            connectionConfig = getWifiConfigurationForNetworkId(mLastInfo.getNetworkId());
        }

        final Collection<ScanResult> results = fetchScanResults();//获取扫描到的网络

        final List<WifiConfiguration> configs = mWifiManager.getConfiguredNetworks();//获取已配置的网络
        if (configs != null) {
            mSavedNetworksExist = configs.size() != 0;
            for (WifiConfiguration config : configs) {
                if (config.selfAdded && config.numAssociation == 0) {
                    continue;//排除 framework 自行添加的网络
                }
                AccessPoint accessPoint = getCachedOrCreate(config, cachedAccessPoints);
              		//如果缓存中存在该网络则直接更新返回，而不是新建
                if (mLastInfo != null && mLastNetworkInfo != null) {
                    accessPoint.update(connectionConfig, mLastInfo, mLastNetworkInfo);
                }
              	//mIncludeSaved，是否需要在结果中显示已配置的网络
              	//这里只是表示不显示配置信息，但该接入点如果被扫描到的话仍会被显示在网络列表中？
                if (mIncludeSaved) {
                    // If saved network not present in scan result then set its Rssi to MAX_VALUE
                    boolean apFound = false;//扫描结果是否包含该网络
                    for (ScanResult result : results) {
                        if (result.SSID.equals(accessPoint.getSsidStr())) {
                            apFound = true;
                            break;
                        }
                    }
                    if (!apFound) {
                        accessPoint.setRssi(Integer.MAX_VALUE);
                    }
                    accessPoints.add(accessPoint);
                    apMap.put(accessPoint.getSsidStr(), accessPoint);
                } else {
                    // If we aren't using saved networks, drop them into the cache so that
                    // we have access to their saved info.
                    cachedAccessPoints.add(accessPoint);
                }
            }
        }
		//对扫描结果进行处理
        if (results != null) {
            for (ScanResult result : results) {
                // Ignore hidden and ad-hoc networks.
                if (result.SSID == null || result.SSID.length() == 0 ||
                        result.capabilities.contains("[IBSS]")) {
                    continue;
                }

                boolean found = false;
                for (AccessPoint accessPoint : apMap.getAll(result.SSID)) {
                    if (accessPoint.update(result)) {
                        found = true;
                        break;
                    }
                }
                if (!found && mIncludeScans) {
                    AccessPoint accessPoint = getCachedOrCreate(result, cachedAccessPoints);
                    if (mLastInfo != null && mLastNetworkInfo != null) {
                        accessPoint.update(connectionConfig, mLastInfo, mLastNetworkInfo);
                    }

                    if (result.isPasspointNetwork()) {
                        // Retrieve a WifiConfiguration for a Passpoint provider that matches
                        // the given ScanResult.  This is used for showing that a given AP
                        // (ScanResult) is available via a Passpoint provider (provider friendly
                        // name).
                        try {
                            WifiConfiguration config = mWifiManager.getMatchingWifiConfig(result);
                            if (config != null) {
                                accessPoint.update(config);
                            }
                        } catch (UnsupportedOperationException e) {
                            // Passpoint not supported on the device.
                        }
                    }

                    accessPoints.add(accessPoint);
                    apMap.put(accessPoint.getSsidStr(), accessPoint);
                }
            }
        }

        // Pre-sort accessPoints to speed preference insertion
        Collections.sort(accessPoints);

        // Log accesspoints that were deleted
        if (DBG) Log.d(TAG, "------ Dumping SSIDs that were not seen on this scan ------");
        for (AccessPoint prevAccessPoint : mAccessPoints) {
            if (prevAccessPoint.getSsid() == null) continue;
            String prevSsid = prevAccessPoint.getSsidStr();
            boolean found = false;
            for (AccessPoint newAccessPoint : accessPoints) {
                if (newAccessPoint.getSsid() != null && newAccessPoint.getSsid().equals(prevSsid)) {
                    found = true;
                    break;
                }
            }
            if (!found)
                if (DBG) Log.d(TAG, "Did not find " + prevSsid + " in this scan");
        }
        if (DBG)  Log.d(TAG, "---- Done dumping SSIDs that were not seen on this scan ----");

        mAccessPoints = accessPoints;
        mMainHandler.sendEmptyMessage(MainHandler.MSG_ACCESS_POINT_CHANGED);
    }
```

#### 获取接入点

​	在网络扫描更新后，WifiTracker 会回调 WifiListener 的 onAccessPointsChanged() 方法，这时候可以调用 WifiTracker 的 getAccessPoints() 方法来获取接入点数据。WifiTracker 还提供了一个 getCurrentAccessPoints(...) 静态方法来强制获取当前的接入点数据。getAccessPoints() 获取的是每一次更新后的数据，而 getCurrentAccessPoints(...) 获取的将是上一次扫描后的数据。

```java
    public static List<AccessPoint> getCurrentAccessPoints(Context context, boolean includeSaved,
            boolean includeScans, boolean includePasspoints) {
        WifiTracker tracker = new WifiTracker(context,
                null, null, includeSaved, includeScans, includePasspoints);
        tracker.forceUpdate();//这里实际调用的是updateAccessPoints()
        return tracker.getAccessPoints();
    }
```



### 监听器

```java
    public interface WifiListener {
      	//disabled、enabled、disabling、enabling、unknown
        void onWifiStateChanged(int state);
      	//连接状态改变，需要调用isConnected来获取更新后的状态
        void onConnectedChanged();
      	//AccessPoints更新，需要调用 getAccessPoints 来获取最新信息
        void onAccessPointsChanged();
    }
```