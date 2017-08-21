---
title: API Guides - 创建输入法
tags:
  - android
  - 输入法
date: 2017-08-19 15:34:30
categories: Android
typora-copy-images-to: ipic
---

首先，了解一下输入法的一些术语：

+ IME：input method editor

+ IMF：input method framework

+ modifier key：修饰键，即组合键，如 shift、ctrl 等

  ​

## 输入法基本知识

一般一个输入法引用需要包括以下部分：

+ 一个继承 InputMethodService 的类
+ 一个用来配置 IME 服务的设置 Activity，我们可以将设置的 UI 显示到系统设置中

 InputMethodService 类是 IME 的核心部分，它除了实现一般的 service 生命周期外，还有各种回调，如提供 IME 的 UI、处理用户输入、将文本传给当前获得焦点的区域。

### IME 生命周期

如下图：

![inputmethod_lifecycle_image](https://ws4.sinaimg.cn/large/006tNc79gy1fip30h8j4xj30bw0nh0tt.jpg)

## 创建 IME

### 在 Manifest 中声明组件

输入法应用需要有一个特殊的 IME 服务，该服务声明时需要指定 `BIND_INPUT_METHOD` 权限并提供一个 action 为 `action.view.InputMethod` 的 intent filter，还需要一个定义了 IME 特征的 metadata。如：

```xml
<!-- Declares the input method service -->
    <service android:name="FastInputIME"
        android:label="@string/fast_input_label"
        android:permission="android.permission.BIND_INPUT_METHOD">
        <intent-filter>
            <action android:name="android.view.InputMethod" />
        </intent-filter>
        <meta-data android:name="android.view.im"
android:resource="@xml/method" />
    </service>
```

还可以提供一个能从系统设置启动的 Activity，它的声明如下：

```java
    <!-- Optional: an activity for controlling the IME settings -->
    <activity android:name="FastInputIMESettings"
        android:label="@string/fast_input_settings">
        <intent-filter>
            <action android:name="android.intent.action.MAIN"/>
        </intent-filter>
    </activity>
```



### 输入法相关 API

IME 相关类主要放在了 `android.inputmethodservice` 和 `android.view.inputmethod` 包中。其中， `KeyEvent` 类是处理键盘字符的一个关键类，而 `InputMethodService` 是IME 的核心部分，`InputMethodService` 提供了大多数管理输入法状态、可见性以及与当前输入区域进行交互的实现。下面两个也是比较重要的类：

+ `BaseInputConnection`

   `InputConnection` 接口的基本实现，可以用来读取光标附近的文字，将文本提交到文本框，还可以将按键事件传给应用。

+ `KeyboardView`

  View 的子类，用于渲染一个键盘并响应用户的输入事件。键盘的布局使用可在 XML 文件中定义的 `Keyboard` 实例来表示。



### 设计输入法 UI

IME 的有两种可见元素：输入视图（input view）和候选视图（candidates view）。



#### 输入视图

输入视图是用户输入文本的地方，常见形式有按键、手写、手势。IME 第一次显示时，系统将调用 [onCreateInputView()](https://developer.android.com/reference/android/inputmethodservice/InputMethodService.html#onCreateInputView()) 回调，在该方法的实现里，需要创建 IME 窗口的显示内容并将该布局返回给系统。如：

```java
@Override
public View onCreateInputView() {
    MyKeyboardView inputView =
        (MyKeyboardView) getLayoutInflater().inflate(R.layout.input, null);

    inputView.setOnKeyboardActionListener(this);
    inputView.setKeyboard(mLatinKeyboard);

    return mInputView;
}
```



#### 候选视图

候选视图用于为用户显示潜在的词语纠正或建议。在 IME 的生命周期中，系统在需要显示候选视图时将调用 [onCreateCandidatesView()](https://developer.android.com/reference/android/inputmethodservice/InputMethodService.html#onCreateCandidatesView()) 。在这个方法的实现中，需要返回用于显示建议词语的布局，如果不需要显示任何东西则直接返回 null。



#### UI 设计注意事项

+ 多屏幕尺寸处理

  适应不同屏幕尺寸、横竖屏处理、非全屏输入模式空出足够空间

+ 处理不同的输入类型（input type）

  如自由格式文本、数字、URL、邮件地址、搜索字符。当我们在实现一个新的 IME 时，需要检测每个区域的输入类型并提供对应的界面。

当输入区域获得焦点且 IME 打开时，系统将调用 onStartInputView() 并传入一个 `EidtorInfo` 对象。

还有一点需要注意的是，当向密码区域传送文本时，要确保文本的正确处理，输入视图和候选视图都要对密码进行隐藏。



### 向应用发送文本

当用户输入文本时，可以通过发送独立按键事件或者通过编辑光标附近文本来向应用发送文本。在这两种情况下，都将使用 InputConnection 实例来传递文本，我们可以通过调用 InputMethodService.getCurrentInputConnection() 来获取该实例。



#### 编辑光标附近文本

当处理这种情况时，BaseInputConnection 提供了一些十分有用的方法：

+ getTextBeforeCursor()
+ getTextAfterCursor()
+ deleteSurroundingText()
+ commitText()



#### 提交前对文本进行合成

如果 IME 可以进行文本预测或者需要多个步骤来合成一个自行或单词，那么可以在文本区域显示演变知道用户提交单词，然后，我们可以使用完成的文本来进行替换。如：

```java
InputConnection ic = getCurrentInputConnection();
ic.setComposingText("Composi", 1);
ic.setComposingText("Composin", 1);
ic.commitText("Composing ", 1);
```

 上面的代码效果如下：

![输入法合成文本](https://ws3.sinaimg.cn/large/006tKfTcgy1fipemv9x1pj30y203smy7.jpg)



#### 拦截物理按键事件

尽管输入法窗口没有明确的焦点，但它将最先接收到物理按键事件并可以选择消耗它们或者转发给应用。

拦截物理按键需要重写 onKeyDown() 和 onKeyUp()。



### 创建 IME 的 subtype

（subtype，子类型？）

subtype 使 IME 可以支持多种输入模式和语言。一个 subtype 表示：

+ 一个 locale 如 en_US 或 fr_FR
+ 一种输入模式如语音、键盘或手写
+ 特定输入法的其他输入样式、形式或属性，如 10-key 或 QWERTY 键盘布局。

我们可以在输入法的 XML 资源文件中使用 <subtype> 元素来定义 subtype。如：

```xml
<input-method xmlns:android="http://schemas.android.com/apk/res/android"
        android:settingsActivity="com.example.softkeyboard.Settings"
        android:icon="@drawable/ime_icon">
    <subtype android:name="@string/display_name_english_keyboard_ime"
            android:icon="@drawable/subtype_icon_english_keyboard_ime"
            android:imeSubtypeLanguage="en_US"
            android:imeSubtypeMode="keyboard"
            android:imeSubtypeExtraValue="somePrivateOption=true" />
    <subtype android:name="@string/display_name_french_keyboard_ime"
            android:icon="@drawable/subtype_icon_french_keyboard_ime"
            android:imeSubtypeLanguage="fr_FR"
            android:imeSubtypeMode="keyboard"
            android:imeSubtypeExtraValue="foobar=30,someInternalOption=false" />
    <subtype android:name="@string/display_name_german_keyboard_ime" ... />
</input-method>
```



### Subtype 切换

我们可以通过提供一个切换键使用户可以轻易地在多种 subtype 间切换，如一个地球状的语言图标。

如果需要启用这样的切换，可以按以下步骤操作：

+ 声明

  ```xml
  <input-method xmlns:android="http://schemas.android.com/apk/res/android"
          android:settingsActivity="com.example.softkeyboard.Settings"
          android:icon="@drawable/ime_icon"
          android:supportsSwitchingToNextInputMethod="true">
  ```

+ 调用 InputMethodManager 的 shouldOfferSwitchingToNextInputMethod() 方法

+ 如果该方法放回 ture，显示切换键

+ 当用户点击切换键时，调用 InputMethodManager 的 switchToNextInputMethod() 方法，它的第二个参数传入 false。false 将使系统同等对待所有的 subtype，不管它们属于哪个 IME；而 true 将使系统只在当前 IME 的 subtype 间进行循环。

需要注意，API 21 之前，switchToNextInputMethod() 并不会理会 supportsSwitchingToNextInputMethod 属性，如果用户切到一个没有切换键的 IME，将被卡在该 IME 而无法切出来。



### 常见 IME 注意事项

+ 提供用户直接从 IME 界面设置配置的方式
+ 提供用户直接从输入法界面切换到不同 IME 的方式，因为用户会在设备中安装多个 IME。（有输入法这么伟大吗。。。）
+ 迅速调出 IME 的 UI。预加载或按需加载所有大的资源，这样用户可以在点击文本区域后立即看到 IME；为随后的输入法调用缓存资源和视图。
+ 相反的，需要在输入法窗口隐藏后释放大的内存占用，这样应用（应该是指其他应用）才有足够的内存来运行。可以考虑在 IME 处于隐藏状态几秒后使用一个延迟消息来释放资源。
+ 确保用户可以为 IME 关联的语言或 locale 输入尽可能多的字符。









参考：

[API Guides - Creating an Input Method](https://developer.android.com/guide/topics/text/creating-input-method.html)

[On-screen Input Methods](https://android-developers.googleblog.com/2009/04/updating-applications-for-on-screen.html)