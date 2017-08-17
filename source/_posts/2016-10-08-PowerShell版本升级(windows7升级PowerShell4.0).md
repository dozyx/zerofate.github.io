---
title: PowerShell - win7下升级
tags:
  -  powershell
date: 2016-10-08 15:37:22
categories: 笔记
---

1. 查看版本 

   > $PSVersionTable


2.  安装**Windows Management Framework 4.0**

     [下载地址](http://www.microsoft.com/zh-CN/download/details.aspx?id=40855) 。window7需要安装sp1，检查是否已安装windows7 SP1 

    > Get-WmiObject -Class Win32_OperatingSystem | Format-Table Caption, ServicePackMajorVersion -AutoSize

3.  安装**.NET Framework 4.5** 

    [下载地址](http://www.microsoft.com/zh-CN/download/details.aspx?id=30653)。检查是否已安装

    > (Get-ItemProperty -Path 'HKLM:\Software\Microsoft\NET Framework Setup\NDP\v4\Full' -ErrorAction SilentlyContinue).Version -like '4.5*'

4.  重启后，查看版本升级成功。