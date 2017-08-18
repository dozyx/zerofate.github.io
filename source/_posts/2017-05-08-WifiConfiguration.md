---
title: WiFiConfiguration 类
tags:
  - android
  - wifi
date: 2017-05-08 15:37:22
categories: 笔记
---

​	WifiConfiguration 用来表示一个已配置的 Wi - Fi 网络，包括安全性配置。



### **KeyMgmt** 静态类

公认的密钥管理方案，即加密类型。KeyMgmt 中定义了以下几种：

+ NONE：未使用WPA，可能为明文或静态WEP
+ WPA_PSK：需要指定preSharedKey
+ WPA_EAP：WPA 使用 EAP 认证。通常是配合一个外部验证服务器使用
+ IEEE8021X：使用EAP认证，并可动态生成WEP密钥（可选）
+ WPA2_PSK：@hide，用于soft access point（软接入点指创建一个网络以便其他设备接入，热点？），需要指定preSharedKey。



### **Protocol** 静态类

公认的安全协议

- WPA：WPA/IEEE 802.11i/D3.0
- RSN：WPA2/IEEE 802.11i



### **AuthAlgorithm** 静态类

公认的 IEEE 802.11 认证算法

+ OPEN：开放系统认证，WPA/WPA2 使用
+ SHARED：共享密钥认证，需要静态的WEP密钥
+ LAEP：LEAP/Network EAP，只能与LEAP使用





### **部分字段**

+ **networkId**：int，客户端用于标识该网络配置条目的id。没有时为**INVALID_NETWORK_ID**。从打印结果上看只是简单的编号，如有6个已配置的wifi，它们的networkId为0到5，测试机器为小米5。**每一个配置过的接入点都有一个有效networkId，包括不需要密码的接入点和密码错误的接入点。**
+ status：int，该网络配置条目的状态，可用值定义在 Status 类中，CURRENT表示该网络为当前连接网络，DISABLED表示客户端不会尝试连接该网络，ENABLE 表示客户端认为该网络可用于连接
+ ==disableReason==：@hide，int，当 status == Status.DISABLED 时，表示 disable 的原因。原因值：DISABLED_UNKNOWN_REASON、DISABLED_DNS_FAILURE、DISABLED_DHCP_FAILURE、DISABLED_AUTH_FAILURE、DISABLED_ASSOCIATION_REJECT、DISABLED_BY_WIFI_MANAGER
+ **SSID**：String，网络 SSID，可以为一个带双引号的 ASCII 字符串，如“MyNetwork”；或者为一个 16 进制数字的字符串
+ BSSID：String，设置后，该网络配置只有在连接的AP具有特定的 BSSID 时才能使用。BSSID 值为 MAC 地址格式：XX:XX:XX:XX:XX:XX。
+ FQDN：String，AAA服务器或RADIUS服务器的域名全称，如“mail.example.com”
+ **preSharedKey**：String，用于WPA-PSK 的 pre-shared 密钥。该值被读取时，并不会返回实际的 key ，在 key 的值存在时将返回 “*”，不存在时返回 null。
+ wepKeys：String[] ，多达四个 WEP 密钥，可以是一个带双引号的 ASCII 字符串或者为一个 16 进制的字符串
+ wepTxKeyIndex：int，默认的 WEP 密钥索引，范围为 0 到 3
+ allowedKeyManagement：BitSet，该配置所支持的密钥管理协议组，值的描述在 KeyMgmt 中，默认支持WPA-PSK和WPA-EAP。BitSet 的set(int index) 方法会将该index位置的值设为true。
+ allowedProtocols：BitSet，该配置支持的安全协议组，值的描述在 Protocol 中。默认支持WPA和RSN。
+ allowedAuthAlgorithms：BitSet，该配置支持的认证协议组，值得描述在 AuthAlgorithm 类中。默认为自动选择。
+ enterpriseConfig：WifiEnterpriseConfig，由 EAP 方法、证书和其他EAP相关设置指定的企业配置细节





### 连接wifi网络

​	WifiManager 提供了两个方法进行连接：

`public void connect(WifiConfiguration config, ActionListener listener)`

`public void connect(int networkId, ActionListener listener)`

















