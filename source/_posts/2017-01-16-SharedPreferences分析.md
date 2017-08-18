---
title: SharedPreferences 使用
tags:
  - android
date: 2017-01-16 15:37:22
categories: 笔记
---

> 对于SharedPreferences文件值的读取和写入原理以及apply和commit的具体实现暂时没有深入分析

## 基本用法

[SharedPreference API Guide](https://developer.android.com/guide/topics/data/data-storage.html#pref)

SharedPreferences实际是将一系列的键值对存储到xml文件。

使用步骤：

1. 获取SharedPreferences对象
2. 读写

### 获取SharedPreferences对象

​	Context提供了[getSharedPreferences(String name, int mode)](https://developer.android.com/reference/android/content/Context.html#getSharedPreferences(java.lang.String, int))方法来获取SharedPreference，该方法接受两个参数，name表示文件名(name.xml)，mode表示文件操作模式，默认为MODE_PRIVATE（0x0000），只允许创建文件的应用访问，文件不存在时会在保存时自行创建。

> 除了MODE_PRIVATE，还有MODE_WORLD_READABLE和MODE_WORLD_WRITEABLE，但这两种模式由于安全性问题已被标记为deprecated。

### 将值存储到文件

将值存储到文件中需要得到SharedPreferences.Editor对象，然后使用Editor的一系列put方法（如 putInt()、putString()等）来保存值，put方法需要传入两个参数，一个键和一个值。最后还需要使用Editor的commit来完成修改。如

```java
SharedPreferences sharedPreferences = getSharedPreferences("name", MODE_PRIVATE);
		Editor editor = sharedPreferences.edit();
		editor.putInt("key_int", 0).putString("key_string", "string");
		editor.commit();
```

> put方法返回的也是一个Editor对象。



### 读取保存的值

SharedPreferences提供了一系列的get方法来获取文件中的值，如getInt()、getString()等，在获取值得时候需要传入一个键和一个默认值。如：

`boolean silent = sharedPreferences.getBoolean("silentMode", false);`



## 原理分析

SharedPreferences实际上是一个Interface，通过Context的getSharedPreferences返回的是其实现类SharedPreferencesImpl的对象。

### 获取SharedPreferences

==ContextImpl.java==

​	SharedPreferencesImpl实例会被保存在sSharedPrefs静态变量中，这里可以看出SharedPreferences在同一进程中是以类似于单例的形式存在的，每个文件名对应了一个实例

` sSharedPrefs = new ArrayMap<String, ArrayMap<String, SharedPreferencesImpl>>();`

​	第一个String是packageName，ArrayMap中的String是getSharedPreferences时传入的name，即文件名。

合成后的文件路径为

`getDataDirFile()/shared_prefs/[name] + ".xml"`

### SharedPreferencesImpl

SharedPreferencesImpl在构造时会先启动现在子线程执行loadFromDiskLocked解析xml文件，解析结果将保存到Map\<String, Object\> mMap中。

### 存储

==SharedPreferencesImpl.java==

put方法是Editor中的方法，在SharedPreferencesImpl中有其实现类EditorImpl。put操作只是将数据存到了mModified变量中

`private final Map<String, Object> mModified = Maps.newHashMap();`

如

```java
public Editor putString(String key, String value) {
            synchronized (this) {
                mModified.put(key, value);
                return this;
            }
        }
```

最终的文件写入是在Editor的commit方法中。

### 读取

==SharedPreferencesImpl.java==

getString源码

```java
public String getString(String key, String defValue) {
        synchronized (this) {
            awaitLoadedLocked();//等待xml文件解析结束
            String v = (String)mMap.get(key);
            return v != null ? v : defValue;
        }
    }
```

==每次的读取都是直接通过mMap从内存读取，而不是读文件==

### commit和apply

[commit](https://developer.android.com/reference/android/content/SharedPreferences.Editor.html#commit())

[apply](https://developer.android.com/reference/android/content/SharedPreferences.Editor.html#apply())

#### commit()

提交该Editor的首选项(preferences)更改，修改成功放回true。当两个editor同时修改了preferences，后一个commit将生效。==如果不需要返回值并且在主线程中提交更改的话，可以考虑使用apply()==。

#### apply()

与commit()功能一致，两个editor同时修改preferences，最后调用apply的editor将生效。



#### 区别与联系

+ commit()会同步地将偏好设置存到持久化存储设备中，而apply()会立即将修改提交到内存，但写入磁盘将会异步进行，并且不会得到任何失败的反馈。如果有另一个editor在apply()执行时对同一个SharedPreferences进行普通的commit()，那么commit()将会被阻塞直到所有的异步提交完成。（即commit会同步执行提交和写入，这可能造成阻塞，而apply会先提交再异步进行写入）

+ 由于SharedPreferences在同一进程内是单例的，所以使用apply()替换所有的commit()是安全的

+ 在不考虑返回值得情况下，优先使用apply()

  ​









