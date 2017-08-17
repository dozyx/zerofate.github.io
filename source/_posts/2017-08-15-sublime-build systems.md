---
title: Sublime Text - Build System
tags:
  - sublime text
date: 2017-08-15 15:37:22
categories: 笔记
---

[Build Systems](http://docs.sublimetext.info/en/latest/reference/build_systems.html)

​	作用：通过构建系统，可以通过外部程序来运行文件，而不需要离开 Sublime Text，并且可以看到它们产生的输出。



## 基础知识

​	简单地 build system 只需要一个 `.sublime-build` 文件，高级点的可能需要多达三个部分组成：

+  `sublime-build` 文件：以 JSON 格式保存的配置数据。
+  可选，用来驱动构建过程的自定义 Sublime Text 命令（Python 代码）。（不懂）
+  可选，外部可执行文件（脚本或二进制文件）。



### .sublime-build 文件格式

+ 格式：JSON
+ 扩展名：.sublime-build
+ 文件名：任意
  + 位置：Package 下任意位置（Package 是 Sublime Text 安装完后文件路径下的一个目录，可以通过 Shift + 	 Command + P 搜索打开查看）
+ Content：预置项和（可选）任意用户定义项



​	`sublime-build` 文件以 JSON 对象保存了配置数据，并指明了开关、选项和环境数据。每个 `.sublime-build` 文件一种文件类型对应的具体范围相关联。

​	示例：

```json
{
    "cmd": ["python", "-u", "$file"],
    "file_regex": "^[ ]*File \"(...*?)\", line ([0-9]*)",
    "selector": "source.python"
}
```



### Build system 中使用的 Sublime Text 命令

​	当在 Sublime Text 中运行（ `Ctrl + B`）默认的构建任务时，Sublime Text 命令将接收 `.sublime-build` 文件中指定的配置数据。然后，该命令将构建文件。通常情况下，它会调起一个外部程序。默认的，build system 中使用的命令是 `exec`，但可以被重写。

> 个人理解：Sublime Text 通过 执行 exec 命令来获取文件的配置数据，然后构建。



### 重写 build system 的默认命令

​	默认的 `exec` 命令的实现位于 `Packages/Default/exec.py`（没找到...）。该命令只是简单地将配置数据转发到外部程序，然后异步运行。

​	使用在 `.build-system` 文件中使用  `target` 选项，可以重写 `exec` 命令。



### 调用外部程序

​	通常，外部程序接收文件或文件夹路径，还有开关和选项来配置它的行为。 





## 配置

有两种主要方式来实现自己的构建系统机制：

+ 作为自定义的 `target` 命令（仍然用的默认的构建系统框架）
+ 作为一个全新的插件（忽略构建系统框架）



### 构建系统的元选项

下面是全部构建系统均能理解的标准选项，这些选项将在 Sublime Text 内部使用。

+ `target`（可选）

  `WindowCommand`，默认为 `exec`（对应 「Packages/Default/exec.py」）。该命令接收 「.sublime-build」 中的所有 target 命令参数。

  用于重写默认的构建系统命令。需要注意的是，如果选择为构建系统重写默认的命令，可以在「.sublime-build」文件中添加任意数量的额外选项。

+ `selector`（可选）

  当 Tools| Build System|Automatic 被选中时使用，Sublime Text 根据 selector 来选中合适的构建系统。

+ `windows`，`osx` 和 `linux`（可选）

  根据系统应用选项。系统指定的值将重写默认值。

+ `variants`（可选）

  （变体？）

  一个列表选项，variant 名将显示在命令板上，这样，在构建系统的 selector 匹配 active 的文件时，就可以轻松地访问。

  使用 variants 可以在同一个 「.sublime-build」文件中指定多个构建系统任务。

+ `name`（可选）

  只在 variant 中有效，用于标识一个构建系统任务。如果 `name` 为「Run」，variant 将显示在 Tools |Build System 下，Sublime Text 将自动将 「Run」任务绑定到 「Ctrl + Shift + B」。



#### target 命令参数

通过 `target` 设置，将可以以选择任意其他命令重写 `exec` 命令。（不懂）



#### 平台特定选项

如：

```json
{
    "cmd": ["ant"],
    "file_regex": "^ *\\[javac\\] (.+):([0-9]+):() (.*)$",
    "working_dir": "${project_path:${folder}}",
    "selector": "source.java",

    "windows": {
        "cmd": ["ant.bat"]
    }
}
```

 在 Windows 下，将使用 「ant.bat」替代 「ant」。



#### variants

如：

```json
{
    "selector": "source.python",
    "cmd": ["date"],

    "variants": [

        { "name": "List Python Files",
          "cmd": ["ls -l *.py"],
          "shell": true
        },

        { "name": "Word Count (current file)",
          "cmd": ["wc", "$file"]
        },

        { "name": "Run",
          "cmd": ["python", "-u", "$file"]
        }
    ]
}
```

「Ctrl + B」将运行 `date` 命令，「Ctrl +Shift + B」 将运行 Python 解析器，并在构建系统被激活时以 `Build:name` 形式将剩下的 variant 显示在命令面板上。



### 捕获构建系统结果

在构建系统将结果输出到结果视图时，可以捕获结果数据以启用结果导航。



如果想要启用结果导航，可以在结果视图中设置以下 view settings：

+ `result_file_regex`

  可以捕获多达 4 个错误信息字段：文件名、行数、列数、错误信息。

+ `result_line_regex`

+ `result_base_dir`



#### 构建系统变量

构建系统支持在 「.sublime-build」中使用以下变量：

$**file_path**	当前文件目录
$**file**		当前文件的完整路径
$**file_name**	当前文件名
$**file_extension**	当前文件扩展名
$**file_base_name**	当前文件名称部分（不包括扩展名）
$**folder**	当前 project 中打开的第一个文件夹的路径
$**project**	当前 project 文件的完整路径
$**project_path**	当前 project 文件的目录
$**project_name**	当前 project 文件名
$**project_extension**	当前 project 文件的扩展名
$**project_base_name**	当前 project 文件的名称部分
$**packages**	 Packages 文件夹的完整路径



## `exec` 命令参数

[链接](http://docs.sublimetext.info/en/latest/reference/build_systems/exec.html)

+ `cmd`
+ `shell_cmd`
+ `working_dir`
+ `encoding`
+ `env`
+ `shell`
+ `path`
+ `file_regex`
+ `line_regex`
+ `syntax`