---
title: Audio Effect 音效
tags:
  - android
  - media
date: 2017-01-11 15:37:22
categories: 笔记
---

[Android Audio代码分析5 - 函数getAudioSessionId](http://blog.csdn.net/njuitjf/article/details/6847408)

[Android Audio代码分析6 - AudioEffect](http://blog.csdn.net/njuitjf/article/details/6848963)

[Issue 22726:	Equalizer on audio session 0 should not be deprecated](https://code.google.com/p/android/issues/detail?id=22726)

[android_atomic_dec android_atomic_inc 实现](http://blog.csdn.net/yujunan/article/details/9020607)

[android的原子操作(x86)](http://www.2cto.com/kf/201110/106982.html)

[原子操作 百度百科](http://baike.baidu.com/link?url=5-13rcIgdB8V1HDmvoFW0-YR6rvy5sA1p8jUadmr_6KWZXPl481-UyLxJK-D5QwKk6HtyWPplE71S8C_VZbb6jp2oW3rPAzD6tQUVx4Y-4G8P3vr76oAA0B11JqOskk1#reference-[1]-809659-wrap)

## 均衡器EQ

[均衡器 百度百科](http://baike.baidu.com/link?url=LxvZal7KDf4zmWdMXFAiClslDdU79p9JM68cOsZvNxHtnVniYK4u2Yse26RvYHAW6eSnfS9Au8v8lzIyDTJgoliWVdGrEdfC8VcHkk2l3yIClNtlsC6dyLSd2Re0tTFY)

### 什么是均衡器

Equalizer，是一种可以**分别调节各种频率成分电信号放大量**的电子设备，通过对各种不同频率的电信号的调节来补偿扬声器和声场的缺陷，补偿和修饰各种声源及其它特殊作用



### 调整范围

**超低音**

20Hz-40Hz，适当时声音强而有力。能控制雷声、低音鼓、管风琴和贝司的声音。过度提升会使音乐变得混浊不清。

**低音**
40Hz-150Hz，是声音的基础部分，其能量占整个音频能量的70%，是表现音乐风格的重要成份。适当时，低音张弛得宜，声音丰满柔和，不足时声音单薄，150Hz，过度提升时会使声音发闷，明亮度下降，鼻音增强。
**中低音**
150Hz-500Hz，是声音的结构部分，人声位于这个位置，不足时，演唱声会被音乐淹没，声音软而无力，适当提升时会感到浑厚有力，提高声音的力度和响度。提升过度时会使低音变得生硬，300Hz处过度提升3-6dB，如再加上混响，则会严重影响声音的清晰度。
**中音**
500Hz-2KHz，包含大多数乐器的低次谐波和泛音，是小军鼓和打击乐器的特征音。适当时声音透彻明亮，不足时声音朦胧。过度提升时会产生类似电话的声音。
**中高音**
2KHz-5KHz，是弦乐的特征音（拉弦乐的弓与弦的摩搡声，弹拔乐的手指触弦的声音某）。不足时声音的穿透力下降，过强时会掩蔽语言音节的识别。
**高音**
7KHz-8KHz，是影响声音层次感的频率。过度提升会使短笛、长笛声音突出，语言的齿音加重和音色发毛。

**极高音**

8KHz-10KHz，合适时，三角铁和立*的金属感通透率高，沙钟的节奏清晰可辨。过度提升会使声音不自然，易烧毁高频单元。



## AudioEffect

AudioEffect是由Android audio framework提供的用于控制音效的基类。应用不应直接使用AudioEffect基类，而是使用其子类来控制指定效果：**Equalizer**、**Virtualizer**、**BassBoost** 、**PresetReverb**、**EnvironmentalReverb**。

==如果需要将音效应用到指定的AudioTrack或MediaPlayer实例，应用必须在创建AudioEffect时指定audio session ID==。

> 注意：通过使用session 0将内嵌效果(insert effect)(equalizer, bass boost, virtualizer)关联到全局的音频输出混音已经被标记为deprecated。（如果主动将MediaPlayer实例的session ID设为0，是否就会对该MediaPlayer有效？）

如果指定的audio session没有相同效果类型的实例存在，将在创建AudioEffect对象的同时创建audio framework中相应的效果引擎；如果已经存在，则使用该实例。

==应用程序创建了一个AudioEffect对象后，能否取得effect enging的控制权，取决于优先级。==如果优先级比正控制effect enging的应用程序的优先级高，则会把控制权抢过来。可以通过设置合适的监听器来监听effect enging的状态或者所有权改变。



## Audio Session ID

​	audio session ID（session：会话）是MediaPlayer实例播放的音频流在系统范围内的唯一标识符。==其主要用途是把音效(audio effect)与特定的MediaPlayer实例进行关联==：如果在创建音效时提供了audio session ID，该效果将仅作用于相同audio session的媒体播放器的音频内容而不是输出混音。

​	创建时，MediaPlayer实例自动生成自己的audio session ID，但是，可以使用setAudioSessionId方法强制将播放器设置为已存在的audio session的一部分(该方法必须在setDataSource方法前调用)。

​	有一个广播`AudioEffect.ACTION_OPEN_AUDIO_EFFECT_CONTROL_SESSION`，根据文档理解似乎是用来通知音效程序对该程序的音频设置音效。



## Equalizer类

​	Equalizer用于改变特定音源或主输出混音的频率响应，其基类是AudioEffect。应用通过创建Equalizer对象来实例化和控制audio framework的Equalizer引擎。应用可以简单地使用预置或者通过equalizer获取每个频段来进行精确控制。

​	使用时可能需要权限：

`<uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS"/>`



### 频段

通过`getNumberOfBands`方法可以获取支持的频段数目，`getBandFreqRange`可以得到指定频段的频率范围，单位为mHz，在一设备中打印出频段如下：

```java
02-05 10:16:17.922: D/DebugActivity(7610): count == 5 & band == 0 & min == 30000 & max == 120000
02-05 10:16:17.922: D/DebugActivity(7610): count == 5 & band == 1 & min == 120001 & max == 460000
02-05 10:16:17.922: D/DebugActivity(7610): count == 5 & band == 2 & min == 460001 & max == 1800000
02-05 10:16:17.922: D/DebugActivity(7610): count == 5 & band == 3 & min == 1800001 & max == 7000000
02-05 10:16:17.922: D/DebugActivity(7610): count == 5 & band == 4 & min == 7000001 & max == 1
```



### 预置效果

通过`getNumberOfPresets`方法可以获取到预置效果的数目，`getPresetName`方法可以获取预置名称，在一设备中按顺序打印出所有预置名称如下：

```java
02-05 10:16:17.922: D/DebugActivity(7610): count == 10 & preset #0 & preset name == Normal
02-05 10:16:17.922: D/DebugActivity(7610): count == 10 & preset #1 & preset name == Classical
02-05 10:16:17.922: D/DebugActivity(7610): count == 10 & preset #2 & preset name == Dance
02-05 10:16:17.922: D/DebugActivity(7610): count == 10 & preset #3 & preset name == Flat
02-05 10:16:17.922: D/DebugActivity(7610): count == 10 & preset #4 & preset name == Folk
02-05 10:16:17.922: D/DebugActivity(7610): count == 10 & preset #5 & preset name == Heavy Metal
02-05 10:16:17.922: D/DebugActivity(7610): count == 10 & preset #6 & preset name == Hip Hop
02-05 10:16:17.922: D/DebugActivity(7610): count == 10 & preset #7 & preset name == Jazz
02-05 10:16:17.922: D/DebugActivity(7610): count == 10 & preset #8 & preset name == Pop
02-05 10:16:17.922: D/DebugActivity(7610): count == 10 & preset #9 & preset name == Rock
```

该android audio framework总共有10种预置。可以通过`usePreset`来直接使用这些预置。





### 自定义效果

` public void setBandLevel(short band, short level)`

​	将给定的增益值设置给给定的equalizer band。`band`为设定新增益的频段，频段编号从0到bands-1；`level`是设置给给定band的新增益，单位为毫贝尔(dB为分贝，如下打印为-15dB~15dB)。`getBandLevelRange()`可以得到可以设置的最大和最小值，在一设备中打印出来如下：

```java
02-05 10:33:34.192: D/DebugActivity(9827): range min == -1500 & range max == 1500
```





## 疑问

### 如何使Equalizer作用于全局

将session id设为0来控制全局的方式已经在文档中表示为deprecated。实际上如果某个MediaPlayer对象使用setAudioSessionId设为0，则仍可以被session 0的Equalizer对象影响。但一般由于不知道第三方播放器使用的session id，所以无法对播放内容的equalizer进行控制。默认的session id通过跟踪源码发现是通过原子操作函数android_atomic_inc（自增）来生成的，个人理解为是对一地址内容进行自增来实现系统唯一性。（**使用一些已发布均衡器等应用却可以对第三方播放器产生影响，不清楚其实现，难道是对更接近底层部分进行了控制？还是直接使用了session 0？自己验证时，session id为0的Equalizer对象可以控制网易云音乐，Android4.4平台**）











