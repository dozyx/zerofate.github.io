---
title: Windows - win10 完全卸载内置 OneDrive
tags:
  - windows
date: 2017-05-06 15:37:22
categories: 笔记
---

[Remove or Uninstall OneDrive from Windows 10 completely](http://www.thewindowsclub.com/uninstall-onedrive-windows-10)

命令行写在：

+ 停止进程

  **TASKKILL /f /im OneDrive.exe**

+ 卸载

  64位 **%systemroot%\SysWOW64\OneDriveSetup.exe /uninstall**

+ 删除剩余目录

  在以下目录 “%UserProfile%, “%LocalAppData% 和 “%ProgramData% 搜索 OneDriver 并删除

+ 删除注册表键（移除文件管理器集成）

  - HKEY_CLASSES_ROOT\Wow6432Node\CLSID\{018D5C66-4533-4307-9B53-224DE2ED1FE6}
  - HKEY_CLASSES_ROOT\CLSID\{018D5C66-4533-4307-9B53-224DE2ED1FE6}

+ 注销

