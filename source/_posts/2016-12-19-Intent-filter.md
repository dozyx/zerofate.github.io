---
title: Intent Filter 分析
tags:
  - android
date: 2016-12-19 15:37:22
categories: 笔记
---

> Intent与intent filter匹配不只是用来发现一个目标component，还可以用来发现设备上的一些component集合，如ACTION_MAIN 和CATEGORY_LAUNCHER。

## 基本语法与作用

```xml
<intent-filter android:icon="drawable resource"
               android:label="string resource"
               android:priority="integer" >
    . . .
</intent-filter>
```

intent-filter用来决定哪个component(activity、service、broadcast receiver)对intent进行响应。

可用于\<activity\> \<activity-alias\>\<service\> \<receiver\>标签，android:priority对activity和broadcast receiver有效：当多个具有不同优先级的activity响应同一个intent时，Android系统只为intent响应高优先级的activity；priority还控制了broadcast receiver接收广播消息的顺序。默认的优先级为0，取值范围为-1000到1000的整数，值越大，优先级越高。

\<intent-filter\>的子标签常用的有action、data、category，可在 [Intent](https://developer.android.com/reference/android/content/Intent.html)查看它们的具体意义。

## action

intent-filter中必须包含一个或多个action，通常使用包名作为action字符串的前缀，如

`<action android:name="com.example.project.TRANSMOGRIFY" />`

## data

[data官方说明](https://developer.android.com/guide/topics/manifest/data-element.html)

为intent-filter添加数据格式，可以只添加数据类型(mimeType)

Uri对象格式：

\<scheme>://\<host>:\<port>[\<path>|\<pathPrefix>|\<pathPattern>]

如content://com.example.project:200/folder/subfolder/etc，folder/subfolder/etc是path。如果scheme没有指定，其余的URI属性将被忽略；如果host没有指定，则剩余部分也会被忽略。同一个\<intent-filter>中的data元素代表了同一个过滤，如

```xml
<intent-filter . . . >
    <data android:scheme="something" android:host="project.example.com" />
    . . .
</intent-filter>
```

和

```xml
<intent-filter . . . >
    <data android:scheme="something" />
    <data android:host="project.example.com" />
    . . .
</intent-filter>
```

是等同的。

语法：

```xml
<data android:scheme="string"
      android:host="string"
      android:port="string"
      android:path="string"
      android:pathPattern="string"
      android:pathPrefix="string"
      android:mimeType="string" />
```

**注意**：代码可以通过setData()方法设置数据URI，setType()设置MIME类型，但如果需要同时设置URI和MIME类型，需要使用setDataAndType()类型而不是同时使用两个上述方法。

## category

字符串，用来表示处理intent的component的类型。

**常见的category**

CATEGORY_BROWSABLE：目标activity为网页浏览器

CATEGORY_LAUNCHER：显示在系统应用启动器的activity

## 匹配规则

系统用三个方面来匹配隐式intent来启动Activity

+ Action
+ Data：URI和数据类型（mime）
+ Category

### action匹配

filter必须包含至少一个action来匹配intent；但**如果一个Intent没有指定action，则具有至少一个action的filter都可以通过匹配**

### category匹配

Intent中的所有category都必须能匹配filter中的category才能通过category检测，而filter中多余的category并不会导致匹配失败。因此，通过一个没有category的Intent总是可以通过匹配，无论filter中声明了什么category。Android会默认为所有的隐式Intent添加 CATEGORY_DEFAULT的category，如果需要activity来处理这些隐式Intent，则需要在<intent-filter>中添加 "android.intent.category.DEFAULT"的category。

### data匹配

Intent和filter中的URI对比，只比较filter有的部分，如

+ filter只指定了指定了scheme，则所有包含相同scheme的URI都匹配
+ filter指定了schema和authority(Authority = Host Name + Port No)但没有path，则所有具有相同scheme和authority的URI通过匹配。
+ filter指定了scheme和authority，则需要同时满足才能匹配

data匹配时需要同时匹配URI和MIME类型，匹配规则

+ intent中	既无URI，也无MIME类型，则只有filter也未指定任何URI或MIME时满足匹配
+ intent包含URI但无MIME，则当filter无MIME且URI匹配时通过
+ intent含有MIME但无URI时，只有filter列出的MIME类型匹配并且filter无URI时通过
+ intent同时包含URI和MIME时，需要同时通过URI和MIME匹配。如果intent的URI为content://或file://，但filter未指定URI，这样通过能通过URI测试，即component在filter只列举了MIME类型时，默认支持content://和file://的data。

对于最后一个规则，如

```xml
<intent-filter>
    <data android:mimeType="image/*" />
    ...
</intent-filter>
```

表示component可以从content provider中获取image数据并显示

```xml
<intent-filter>
    <data android:scheme="http" android:type="video/*" />
    ...
</intent-filter>
```

表示component可以从网络上获取视频数据



参考资料

[官网intent-filter](https://developer.android.com/guide/topics/manifest/intent-filter-element.html)

[Android学习之——intent-fliter配置之data属性](http://blog.csdn.net/csxwc/article/details/10222913)

