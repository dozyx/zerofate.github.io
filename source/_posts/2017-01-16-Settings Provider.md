---
title: Settings Provider 的使用
tags:
  - android
date: 2017-01-16 15:37:22
categories: 笔记
---

​	Settings provider 用来存储全局的系统级别的设备偏好设置，该 provider 的读写需要通过 Settings 类，Settings 类提供了一系列的 putXX() 和 getXX() 静态方法来访问 Settings provider。

​	在 Settings 类中又定义了三个访问权限不同的静态内部类来对设置项进行分类存储，这三个类表示了三张表。上面所说的 putXX() 和 getXX() 也需要通过这些内部类才能使用：

+ Settings.Global

  全局系统设置，对所有已确定用户生效。应用可读但不可写，用户只能通过系统 UI 或专门的 API 来显式修改这些设置。 如：时间/时区自动更新。

+ Settings.System

  系统设置，提供了访问用户独立设置项的方法。

+ Settings.Secure

  安全系统设置，与 Settings.Global 类似，应用可读但不可写，用户只能通过系统 UI 或专门的 API 来显式修改这些设置。 

在这三个类中分别定义了它们存储的字段。