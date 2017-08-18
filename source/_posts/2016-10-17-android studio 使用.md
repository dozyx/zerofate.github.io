---
title: Andoid Studio - 使用
tags:
  - android
date: 2016-10-17 15:37:22
categories: 笔记
---

### Gradle配置

[官网](https://developer.android.google.cn/studio/build/index.html#build-config)

### 添加依赖 ###

1. 从远程仓库中下载引用：File>>Project Structure选择module的dependcies选项
2. 将第三方jar包作为依赖：将jar放到app>>src>>main>>libs目录下，右键jar包选择Add As Library
3. 自己在module的build.gradle文件中修改dependencies

### 将github项目导入android studio ###
[参考](http://www.cnblogs.com/Sharley/p/5519053.html)
1. 设置Git程序路径:File >> Settings >> Version Control >> Git
2. 添加Github账号:File >> Settings >> Version Control >> Git
3. Checkout:VCS >>Checkout from Version Control >> Github

File >> Settings >> Version Control设置项目的VCS管理工具，这样在工具栏VCS和工具栏中会出现一系列的操作选项





### 将project上传到github

1. File >> Settings >> Version Control Github

   配置github账户（也可以在下一步中根据提示设置）

2. VCS >> Import into Version Control >> Share Project on Github

   根据提示上传创建repository（需要填写repository描述和commit信息，第一次commit可以填写initial commit）



### 导入Android Studio工程

​	使用本地gradle：新建工程，将工程中的gradle文件夹和build.gradle文件替换导入工程中的相应文件，再进行导入。（未成功验证）



### 小技巧

#### 快速生成TAG

onCreate()方法外面输入logt，然后Tab，这时将会以当前的类名自动生成一个TAG常量。



#### 修改或删除类抬头的文档说明

Settings -> Editor -> File and Code Templates

修改：

Includes -> File Header

删除：

Files ->Class -> 删除 #parse("File Header.java")

### 快捷键

#### shift+shift

搜索源代码、数据库、操作和用户界面的元素等



#### ctrl + E

最近文件



#### ctrl + shift + L

格式化



#### ctrl + shift + u

大小写切换



#### ctrl + i

生成接口的实现方法



#### alt + insert

生成方法



#### ctrl + w

选择当前作用域所有内容



#### ctrl + u

查看父类方法



#### 自定义

+ **alt + shift + 1** 新建java class
+ **alt + shift + 2** 新建布局资源文件
+ **alt + shift + 3** 新建值资源文件