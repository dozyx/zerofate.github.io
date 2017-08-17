---
title: sdkmanager 命令
tags:
  - android
date: 2017-03-25 15:37:22
categories: 笔记
---

## sdkmanager

​	sdkmanager是一个命令行工具，可以用来查看、安装、更新、卸载 Android SDK 中的包。  
> ​	其实，如果安装了 Android Studio 的话，可以直接在IDE中完成这些工作。本人接触到这个命令行，是因为IDE下载包失败后怀疑是代理问题，于是想要通过 sdk manager 设置镜像源进行更新，但发现新的 sdk 中已经没有单独的 sdk manager ，并且 android 命令行工具也已经废弃。可能用到 sdkmanager 命令行的机会并不多，此记录仅为了备忘。

​	sdkmanager 在 Android SDK Tools 包的 25.2.3 版本开始提供，位于 *android_sdk/tools/bin/* 目录下。

### 用法

#### 列出已安装和可用的包
`sdkmanager --list [options]`



#### 安装包

**安装**

`sdkmanager packages [options]`

​	*packages* 参数可通过 *--list* 命令查看，并且需要使用双引号包含起来，如 "build-tools;25.0.2" 或者 "platforms;android-25"，可使用空格区分多个包。
示例：
`sdkmanager "platforms;android-25"`



​	还可以使用一个包含了指定包的text文件来进行安装
`sdkmanager --package_file=package_file [options]`
​	其中，*package_file* 参数是一个本地text文件，其中每一行是一个包，包的名称不需要使用双引号。

**卸载**

`sdkmanager --uninstall packages [options]`
`sdkmanager --uninstall --package_file=package_file [option`



#### 更新所有安装的包

`sdkmanager --update [options]`



### Options

| Option                                   | Description                              |
| ---------------------------------------- | ---------------------------------------- |
| --sdk_root=**path**                      | 使用指定路径的SDK而不是包含此工具的SDK                   |
| --channel=**channel_id**                 | 指定包的渠道， 可用的有:`0` (Stable), `1` (Beta), `2` (Dev), and `3` (Canary). |
| --include_obsolete                       | 在列表和更新时包含已过时的包，只在`--list` 和 `--update` 中可用。 |
| --no_https                               | 强制所有连接使用http而不是https。                    |
| --verbose                                | Verbose输出模式。错误、警告和信息级别的消息都会打印出来。         |
| --proxy={http \| socks}                  | 代理连接时的类型， `http` 用于高级协议如 HTTP or FTP， `socks用于 SOCKS (V4 或 V5) 代理。 |
| --proxy_host={**IP_address** \|**DNS_address**} | 代理使用的IP或域名地址。                            |
| --proxy_port=**port_number**             | 代理连接的端口。                                 |

> 注意：设置代理时，需要同时指定类型、地址、端口。