---
title: imeOptions 属性
tags:
  - android
  - 输入法
date: 2017-05-02 15:37:22
categories: 笔记
---

​	为与Editor关联的 IME 提供额外特性，用于提高应用与输入法的交互。`android:imeOptions`与`android.view.inputmethod.EditorInfo#imeOptions`对应。

​	actionXXX 表示 action 键所执行的动作。



+ normal

+ actionUnspecified

  editor没有指定的action与之关联，将由 editor 自行决定

+ actionNone

  editor没有关联的action（全屏状态将不在右侧显示action按钮）

+ actionGo

  如用户输入URL

+ actionSearch

+ actionSend

+ actionNext

  进入下一需要接收文本的字段

+ anctionDone

+ actionPrevious

+ flagNoFullscreen

  要求 IME 永远不要进入全屏模式（注意：该 flag 并不一定对所有 IME 有效）

+ flagNavigatePrevious

+ flagNavigateNext

  表明存在可用于向前导航进行聚焦的控件。与actionNext 类似，除了可以允许 IME 可以有多行，包括一个 enter 按键和一个向前导航。注意：可能有的IME 并不允许此操作，尤其是在空间有限的小屏幕，在这种情况下，将不会为此选项增加显示 UI。

+ flagNoExtractUI

  表明IME 不需要显示额外的文本UI（即在全屏时额外的文本编辑框和action按钮）。对于可能全屏（通常在横屏状态）的输入法，这将允许输入法变得更小并且让应用的一部分通过全屏IME的透明部分在后面显示。（设置该flag后，在输入法弹出时，Activity界面将上移，编辑框的baseline位于输入法上边界。设置了该属性后，**IME仍可以进入全屏，但输入法以上的视图处于透明状态**）。可见的UI部分可能无法处理touch事件，因为IME将接收touch事件，因此，使用IME_FLAG_NO_FULLSCREEN替换此flag可能带来更好的体验。**该flag并不建议使用，并可能在以后被标记为过时。**

+ flagNoAccessoryAction

  表明IME 全屏模式下，action 不需要在右侧显示为附加按钮。

+ flagNoEnterAction

+ flagForceAscii



























