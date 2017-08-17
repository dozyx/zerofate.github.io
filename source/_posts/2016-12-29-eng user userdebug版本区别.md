---
title: eng、user、userdubug 版本区别
tags:
  - android
date: 2016-12-29 15:37:22
categories: 笔记
---

[Preparing to Build](https://source.android.com/source/building.html)

[The difference between Android source code compiler options eng, user, userdebug](http://www.programering.com/a/MTO0gzNwATk.html)

\build\core\build-system.html

通常使用lunch来build target，如

> $ lunch aosp_arm-eng	

build target的格式为BUILD_BUILDTYPE，其中BUILD是代码名称，BUILDTYPE可以取以下值：

| Buildtype | Use                                      |
| --------- | ---------------------------------------- |
| user      | limited access; suited for production    |
| userdebug | like "user" but with root access and debuggability; preferred for debugging |
| eng       | development configuration with additional debugging tools |

## 区别

### **eng**

This is the default flavor.A plain "make" is the same as "make eng". droid is an alias for eng.	

+ Installs modules tagged with:`eng`, `debug`, `user`, and/or `development`.
+ Installs non-APK modules that have no tags specified.
+ Installs APKs according to the product definition files, in addition to tagged APKs.
+ `ro.secure=0`
+ `ro.debuggable=1`
+ `ro.kernel.android.checkjni=1`
+ `adb` is enabled by default.

### **user**

This is the flavor intended to be the final release bits.

+ Installs modules tagged with `user`.
+ Installs non-APK modules that have no tags specified.
+ Installs APKs according to the product definition files; tags are ignored for APK modules.
+ `ro.adb.secure=1`
+ `ro.secure=1`
+ `ro.debuggable=0`
+ `adb` is disabled by default.

### **userdebug**

The same as `user`, except:

+ Also installs modules tagged with `debug`.
+ `ro.debuggable=1`
+ `adb` is enabled by default.




源码一般在Android.mk文件中配置编译选项，一般形式为

> LOCAL_MODULE_TAGS := user eng optional test

optional表示所有的编译版本都会将该module编译进去。