---
title: Gradle - BuildConfig
tags:
  - gradle
date: 2017-07-18 15:37:22
categories: 笔记
---

[ANDROID BUILDCONFIG.DEBUG的妙用](http://stormzhang.com/android/2013/08/28/android-use-build-config/)

[Gradle Android Plugin 中文手册——BuildConfig](https://chaosleong.gitbooks.io/gradle-for-android/content/build_variants/buildconfig.html)

[Simplify app development](https://developer.android.com/studio/build/gradle-tips.html#simplify-app-development)

​	在编译时，Gradle 会**自动生成**一个名为 BuildConfig 的类(从参考中似乎 Eclipse 的 ADT 也会生成，但未验证)，该类包含了当前编译的信息。

​	比如，一个 BuildConfig.java 可能如下：

```java
/**
 * Automatically generated file. DO NOT MODIFY
 */
package com.example.android.displayingbitmaps;

public final class BuildConfig {
  public static final boolean DEBUG = Boolean.parseBoolean("true");
  public static final String APPLICATION_ID = "com.example.android.displayingbitmaps";
  public static final String BUILD_TYPE = "debug";
  public static final String FLAVOR = "";
  public static final int VERSION_CODE = 1;
  public static final String VERSION_NAME = "1.0";
}
```

​	根据编译配置的不同将导致这些变量发生变化，如将 Build Variant 改为 release，然后在 AS 中选择 Build -> Clean Project(或者 Rebuild Project)，对应的 DEBUG 变量的值将变为 false，BUILD_TYPE 的值将变为 “release”。

​	我们可以利用这些变量来区分不同的变体从而改变其行为，如根据 DEBUG 的值来判断是否输出 LOG，这样就可以直接使用 IDE 自动生成的值，而不需要在编译前主动修改。



### 自定义 BuildConfig 字段

​	除了使用原有字段外，我们也可以在 Gradle 编译配置文件中使用 `buildConfigField()` 方法来为 BuildConfig 类添加自定义字段。类似的，可以使用 `resValue()` 来添加资源值。

```groovy
android {
  ...
  buildTypes {
    release {
      // These values are defined only for the release build, which
      // is typically used for full builds and continuous builds.
      buildConfigField("String", "BUILD_TIME", "\"${minutesSinceEpoch}\"")
      resValue("string", "build_time", "${minutesSinceEpoch}")
      ...
    }
    debug {
      // Use static values for incremental builds to ensure that
      // resource files and BuildConfig aren't rebuilt with each run.
      // If they were dynamic, they would prevent certain benefits of
      // Instant Run as well as Gradle UP-TO-DATE checks.
      buildConfigField("String", "BUILD_TIME", "\"0\"")
      resValue("string", "build_time", "0")
    }
  }
}
...
```

​	然后，通过 BuildConfig 使用

```java
...
Log.i(TAG, BuildConfig.BUILD_TIME);
Log.i(TAG, getString(R.string.build_time));

```



### 与 manifest 共享属性

​	某些情况下，可能需要在 Manifest 和 代码中声明同一个属性，为避免在多处修改同一变化，可以在 module 的 build.gradle 中定义属性，这样 manifest 和代码都可用。如：

定义

```groovy
android {
  // For settings specific to a product flavor, configure these properties
  // for each flavor in the productFlavors block.
  defaultConfig {
    // Creates a property for the FileProvider authority.
    def filesAuthorityValue = applicationId + ".files"
    // Creates a placeholder property to use in the manifest.
    manifestPlaceholders =
      [filesAuthority: filesAuthorityValue]
      // Adds a new field for the authority to the BuildConfig class.
      buildConfigField("String",
                       "FILES_AUTHORITY",
                       "\"${filesAuthorityValue}\"")
  }
  ...
}
...
```

manifest 中访问

```xml
<manifest>
  ...
  <application>
    ...
    <provider
      android:name="android.support.v4.content.FileProvider"
      android:authorities="${filesAuthority}"
      android:exported="false"
      android:grantUriPermissions="true">
      ...
    </provider>
  </application>
</manifest>
```

代码访问

```java
...
Uri contentUri = FileProvider.getUriForFile(getContext(),
  BuildConfig.FILES_AUTHORITY,
  myFile);
```