---
title: activity-alias 标签
tags:
  - android
date: 2017-03-24 15:37:22
categories: 笔记
---

[activity-alias](https://developer.android.com/guide/topics/manifest/activity-alias-element.html)



### 语法

```java
<activity-alias android:enabled=["true" | "false"]
                android:exported=["true" | "false"]
                android:icon="drawable resource"
                android:label="string resource"
                android:name="string"
                android:permission="string"
                android:targetActivity="string" >
    . . .
</activity-alias>
```

包含于\<application\>标签，可以包含\<intent-filter\>和\<meta-data\>标签。



### 描述

​	为*targetActivity*指定的Activity设置别名，该target必须与别名Activity存在于同一个应用中，并且在清单中必须声明在别名之前。

​	alias作为一个单独的实体，代表了target activity。它可以有自己的intent filter而不是target activity中的intent filter。如，alias中可以指定 "android.intent.action.MAIN" 和"android.intent.category.LAUNCHER" 标记，这样将导致alias activity显示在应用的launcher中，即使target activity的filter中没有这样的标记。



​	除targetActivity外，\<activity-alias\>中的属性均为\<activity\>属性的一个子集，对于子集中有的属性，target所设置的值不会传递给alias，但是，对于不在子集中的属性，target activity设置的值将应用到alias中。



### 属性

android:enabled

​	系统是否可以通过该别名对target activity进行实例化。\<application\>有自己的*enabled*属性，并且只有当两个地方的值都为true时，系统才能通过别名将target activity实例化。



android:exported

​	其他应用的component是否可以通过别名启动target activity。如果为*false*，则只有同一应用或同一user ID的应用呃component可以通过别名来启动target activity。

​	==默认值取决于alias是否包含intent filter。==



android:icon

android:label



### 使用场景

+ 在部署app时，launcher activity的清单入口将作为一个合约。当包名或类名发生改动时，将可能造成困扰，通过设置一个别名，可以避免这种麻烦。==alias指定的name并不需要必须存在。==
+ 为同一Activity设置多个Launcher入口，如不同的label、icon但启动同一个Activity，并且可以通过不同的conponent name进行区分。