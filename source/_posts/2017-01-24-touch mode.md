---
title: 什么是 touch mode
tags:
  - android
date: 2017-01-24 15:37:22
categories: 笔记
---

[Touch Mode](https://android-developers.googleblog.com/2008/12/touch-mode.html)

[What you should know about Android Touch Mode and Focus](http://www.101apps.co.za/index.php/articles/what-you-should-know-about-android-touch-mode-and-focus.html)

> 以下内容基本摘自google博客文章



> The touch mode is a state of the view hierarchy that depends solely on the user interaction with the phone. touch mode是视图层级依赖于用户与手机交互方式的一种状态。

touch mode简单指示了用户上一次交互是否使用了触摸。如，对于G1手机，通过轨迹球选择一个widget将使用户脱离touch mode；但如果用手机触摸了屏幕上的button又将进入touch mode。除了touch mode，还有trackball mode，navigation mode，keyboard navigation。

> 设备会保持在touch mode直到用户退出touch mode



==在touch mode下，并不存在focus和selection。==一旦进入touch mode，list或者grid中任意处于selected的item将变成unselected。类似地，在进入touch mode时，任意的focused widgets也将变成unfocused。当用户离开touch mode，framework会复原selection/focus。touch mode与selection和focus间的这种关系意味着不应该依靠selection或者focus来退出应用。对于新手开发者，一个非常普遍的问题是依靠ListView.getSelectedItemPosition()，在touch mode，该方法返回的是INVALID_POSITION，我们应该使用chick监听器或者choice模式来替代。

==事实上，focus在touch mode中不存在是不完全正确的，有一种方式可以让foucus存在于touch mode中，即focusable in touch mode。==这种特殊模式是为了接受文本输入的widget而创建的，如EditText和允许过滤的ListView。

> focusable in touch mode应该谨慎使用，因为这可能导致与Android普通行为的不一致性。

新手开发者通常以为focusable in touch mode是“解决”selection/focus消失的方法，但在使用之前必须深思熟虑。不恰当的使用将导致应用与系统其他部分不一致并可能脱离用户习惯。Android framework包含了所有不使用“focusable in touch 摸得”来解决用户交互问题的工具，如，使ListView保持selection，可以适当地使用choice mode。

**touch mode备忘单**

该：

+ 保持核心应用的连贯性
+ 在需要持久化selection时使用合适的特征（radio button，check box，ListView的choice mode等）
+ 如果是在编写一个游戏，可以使用focusable in touch mode

不该：

+ 不要尝试在touch mode时保留focus和selection	









