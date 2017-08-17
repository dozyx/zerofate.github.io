---
title: Sublime Text - 直接运行 Java 代码
tags:
  - sublime text
date: 2017-08-15 15:37:22
categories: 笔记
---

[How to run Java using Sublime Text 3 on Mac OS](https://stackoverflow.com/questions/24319143/how-to-run-java-using-sublime-text-3-on-mac-os)



> 在 Sublime Text 中编写一个代码文件，然后按下 Ctrl + B，将会对文件进行编译，但并不会显示出运行结果，如一个 Hello.java 文件，Ctrl + B，将自动编译为一个 Hello.class 文件，Sublime Text 只会输出 “[Finished in XXs]”。 为了实现代码的运行，需要对修改对应的配置文件。下面以 Java 文件为例。



Sublime Text 构建系统的配置数据保存在 .sublime-build 后缀文件中，

先打开 Packages

> $ cd /Applications/Sublime\ Text.app/Contents/MacOS/Packages/

创建一个临时目录

> $ mkdir java

将 「Java.sublime-package」复制到临时目录中

> $ cp Java.sublime-package java/
>
> $ cd java

解压 「Java.sublime-package」

> $ unzip Java.sublime-package

接下来编辑  JavaC.sublime-build 配置文件

将里面的内容改为

```json
{
    "cmd": ["javac \"$file_name\" && java \"$file_base_name\""],
    "shell": true,
    "file_regex": "^(...*?):([0-9]*):?([0-9]*)",
    "selector": "source.java"
}
```

（主要就是在 javac 后面加了 java 部分）

最后压缩文件并替换原来的文件

> $ zip Java.sublime-package *
>
> $ mv Java.sublime-package ../

删除临时目录

> $ cd ..
>
> $ rm -fr java/

现在，重启 Sublime Text，重新运行代码，就可以直接在 Sublime Text 中看到输出。

