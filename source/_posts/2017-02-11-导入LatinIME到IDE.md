---
title: 如何将源码中的 LatinIME 导入 Eclipse
tags:
  - android
  - 输入法
date: 2017-02-11 15:37:22
categories: 笔记
---

### Eclipse导入原生LatinIME

[Setup AOSP keyboard](http://yyl.github.io/android/2014/06/01/aosp-keyboard-setup.html)

（1）导入an existing project
（2）添加支持库：eclipse，复制支持库的jar到libs
（3）在sdk源码中找到com.android.inputmethodcommon的包，复制进工程中
（4）将java-overridable文件夹下的源文件合并到工程中
（5）提高sdk版本(此时编译器已无错误)
（6）运行时报错，将报错的资源文件后缀.dict改为.jet（具体原因不明）


