---
title: AS报错：MissingTranslation
tags:
  - android
date: 2017-03-19 15:37:22
categories: 笔记
---

有好几种方式解决翻译缺失的问题

+ 在 string 标签中指定 translatable，在 AS 的“Translations Editor” 中可以快速将一个字符串标记为untranslatable

  `<string name="hello" translatable="false">hello</string>`

+ 在 resource 中添加 ignore 属性

  ```
  <resources
    xmlns:tools="http://schemas.android.com/tools"
    tools:ignore="MissingTranslation" >

    <!-- your strings here; no need now for the translatable attribute -->

  </resources>
  ```

+ 在 build.gradle 中配置lint

  ```groovy
  android {
       lintOptions {
          disable 'MissingTranslation'
      }
  }
  ```
  如果需要禁用多项，以逗号分隔，如

  ```groovy
  android {
       lintOptions {
          disable 'MissingTranslation','ExtraTranslation'
      }
  }
  ```

+ 创建一个前缀为*donottranslate*的资源文件专门放置不需要翻译的字符串，如donottranslate.xml，donottranslate_url.xml