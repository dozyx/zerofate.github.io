---
title: Mac - 下载 Youtube 视频
tags:
  - mac
  - youtube
date: 2017-06-04 15:37:22
categories: 笔记
---

### youtube-dl

[官方](https://rg3.github.io/youtube-dl/download.html)

#### 安装 youtube-dl

可以选择多种方法

1. 需要已安装Homebrew（已试）

   `brew install youtube-dl`

2. pip 方式

   `sudo pip install --upgrade youtube_dl`

3. curl方式

   ```
   sudo curl -L https://yt-dl.org/downloads/latest/youtube-dl -o /usr/local/bin/youtube-dl
   sudo chmod a+rx /usr/local/bin/youtube-dl
   ```

4. wget方式

   ```
   sudo wget https://yt-dl.org/downloads/latest/youtube-dl -O /usr/local/bin/youtube-dl
   sudo chmod a+rx /usr/local/bin/youtube-dl
   ```

#### 开始下载

`youtube-dl --proxy socks5://127.0.0.1:1080/ <视频地址>`

此处使用的是代理是 shadowsocks，如果是在 win 环境下的 shadowsocks，应该改为 localhost:1080，不过没有验证。



#### youtube-dl 可选参数

`--list-formats` 

查看可用画质和格式

`-f <format_code>`

下载指定画质与格式的视频