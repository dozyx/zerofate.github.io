---
title: Mac - Android 开发的一些问题
tags:
  - andorid
date: 2017-05-10 15:37:22
categories: 笔记
---

### 配置环境变量使用adb命令

打开$HOME/.bash_profile文件，添加

export PATH=${PATH}:/Users/zero/Library/Android/sdk/platform-tools
export PATH=${PATH}:/Users/zero/Library/Android/sdk/tools

（注意：zero为mac的用户名）

在终端输入

source .bash_profile