---
title: anim 和 animator
tags:
  - android
  - 动画
date: 2017-03-30 15:37:22
categories: 笔记	
---

在res目录下可以添加两个动画相关的目录，一个是anim，一个是animator。

anim主要存放的view动画，而animator中存放的是属性动画。在framework中，动画类主要在两个包中：android.view.animation和android.animation，前者为view动画，其中有许多类名中含有animation，后者为属性动画，其中又许多类名中函数animator。