---
title: PowerShell - 配置
tags:
  - powershell
date: 2016-12-02 15:37:22
categories: 笔记
---

### 修改默认目录

通过自定义profile脚本文件修改。

1. 检查是否存在profile文件

   > Test-Path $profile

2. 上一步中命令如果返回False表示文件不存在，需要新建新的profile，如果返回True，可跳过此步。

   > New-Item -path $profile -type file –force

3. 根据创建时提示的目录找到脚本文件，打开

   如目录为 C:\Users\用户名\Documents\WindowsPowerShell

4. 目录新建后内容为空，使用set-location设置根目录

   如设置为d盘：set-location d:



### 属性设置

修改窗口名字 

> $Shell.WindowTitle=”SysadminGeek” 

修改窗口大小和滚动 

> \$Shell = $Host.UI.RawUI 

> \$size = $Shell.WindowSize

> $size.width=70

> $size.height=25

> \$Shell.WindowSize = $size

> \$size = $Shell.BufferSize 

> $size.width=70 

> $size.height=5000 

> \$Shell.BufferSize = $size 

修改颜色 

> \$shell.BackgroundColor = “Gray” 

> $shell.ForegroundColor = “Black” 

添加别名 

> new-item alias:np -value C:\Windows\System32\notepad.exe 

清空屏幕缓存 

> Clear-Host





### 问题

#### 在修改profile后重新启动PowerShell提示禁止执行脚本

执行

> set-ExecutionPolicy RemoteSigned

修改策略（需要管理员权限）



