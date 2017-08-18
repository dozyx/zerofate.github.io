---
title: 在 Android Studio 项目中使用自己的 framework
tags:
  - android
date: 2017-04-10 15:37:22
categories: 笔记
---

添加自己framework包的依赖

```groovy
provided files('systemlibs/framework.jar')
```

在gradle中添加如下代码

**注意修改jar包路径**

```groovy
allprojects {     
	gradle.projectsEvaluated {         
		tasks.withType(JavaCompile) {             
			options.compilerArgs.add('-Xbootclasspath/p:app/systemlibs/framework.jar')         
		}     
	}     
	repositories {         
		jcenter()     
	} 
}
```

> add 中如果有多个 jar 包则使用分号分隔。

然后在app.iml 中将framework库放到 Android api 的前面，这样可以优先调用自己的 framework 中的api，如

```groovy
<orderEntry type="library" exported="" name="framework" level="project" />     
<orderEntry type="jdk" jdkName="Android API 25 Platform" jdkType="Android SDK" />
```

> 注意：手动改变orderEntry后，每次sync都会恢复默认顺序。

可以在module 的gradle下添加以下代码自动将 SDK 置后

```groovy
preBuild {
    doLast {
        def imlFile = file(project.name + ".iml")
        println 'Change ' + project.name + '.iml order'
        try {
            def parsedXml = (new XmlParser()).parse(imlFile)
            def jdkNode = parsedXml.component[1].orderEntry.find { it.'@type' == 'jdk' }
            parsedXml.component[1].remove(jdkNode)
            def sdkString = "Android API " + android.compileSdkVersion.substring("android-".length()) + " Platform"
            new Node(parsedXml.component[1], 'orderEntry', ['type': 'jdk', 'jdkName': sdkString, 'jdkType': 'Android SDK'])
            groovy.xml.XmlUtil.serialize(parsedXml, new FileOutputStream(imlFile))
        } catch (FileNotFoundException e) {
            // nop, iml not found
        }
    }
}
```

>  这里的 sdkString 也可以手动替换为固定字符串，如"Android API 25 Platform (1)"，不知道为什么我的sdk 字符串后面有个"(1)"，而sdkString拼接的没有，导致运行时没找到sdk。