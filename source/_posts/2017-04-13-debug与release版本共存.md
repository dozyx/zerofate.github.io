---
title: 为 debug 和 release 指定不同包名
tags:
  - android
date: 2017-04-13 15:37:22
categories: 笔记
---

[Android多版本共存-基于gradle实现debug版和release版app共存](http://www.jianshu.com/p/3724533dcd6a)



配置 module 的build.gradle 文件

```groovy
android{
        ...
        buildTypes{
            debug{
                //在编译打包时会给包名加上后缀
                applicationIdSuffix'.debug'
            }
            release{

            }
        }
        ...
    }
```



打开Build Variants 界面，设置 module 的编译版本![debug与release共存](C:\Users\sdt13599\OneDrive\markdown\photo\AS\debug与release共存.png)