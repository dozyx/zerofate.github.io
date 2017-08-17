---
title: buttonStyle
tags:
  - android
  - style
date: 2017-01-06 15:37:22
categories: 笔记
---

## Holo

```xml
    <style name="Widget.Holo.Button" parent="Widget.Button">
        <item name="background">@drawable/btn_default_holo_dark</item>
        <item name="textAppearance">?attr/textAppearanceMedium</item>
        <item name="textColor">@color/primary_text_holo_dark</item>
        <item name="minHeight">48dip</item>
        <item name="minWidth">64dip</item>
    </style>
```

btn_default_holo_dark.xml

```xml
//这里用的drawable都是图片
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:state_window_focused="false" android:state_enabled="true"
        android:drawable="@drawable/btn_default_normal_holo_dark" />
    <item android:state_window_focused="false" android:state_enabled="false"
        android:drawable="@drawable/btn_default_disabled_holo_dark" />
    <item android:state_pressed="true" 
        android:drawable="@drawable/btn_default_pressed_holo_dark" />
    <item android:state_focused="true" android:state_enabled="true"
        android:drawable="@drawable/btn_default_focused_holo_dark" />
    <item android:state_enabled="true"
        android:drawable="@drawable/btn_default_normal_holo_dark" />
    <item android:state_focused="true"
        android:drawable="@drawable/btn_default_disabled_focused_holo_dark" />
    <item
         android:drawable="@drawable/btn_default_disabled_holo_dark" />
</selector>
```



## DeviceDefault与Material

这两个theme中的buttonStyle都是同一个

```xml
<style name="Widget.Material.Button">
        <item name="background">@drawable/btn_default_material</item>
        <item name="textAppearance">?attr/textAppearanceButton</item>
        <item name="minHeight">48dip</item>
        <item name="minWidth">88dip</item>
        <item name="stateListAnimator">@anim/button_state_list_anim_material</item>
        <item name="focusable">true</item>
        <item name="clickable">true</item>
        <item name="gravity">center_vertical|center_horizontal</item>
    </style>
```

btn_default_material.xml

```xml
<!-- 这里的color定义的是波纹的颜色，即波纹基于什么颜色变化 -->
<!-- item定义的是button的外观 -->
<ripple xmlns:android="http://schemas.android.com/apk/res/android"
        android:color="?attr/colorControlHighlight">
    <item android:drawable="@drawable/btn_default_mtrl_shape" />
</ripple>
```

btn_default_mtrl_shape.xml

```xml
<inset xmlns:android="http://schemas.android.com/apk/res/android"
       android:insetLeft="@dimen/button_inset_horizontal_material"
       android:insetTop="@dimen/button_inset_vertical_material"
       android:insetRight="@dimen/button_inset_horizontal_material"
       android:insetBottom="@dimen/button_inset_vertical_material">
    <shape android:shape="rectangle"
           android:tint="?attr/colorButtonNormal">
        <corners android:radius="@dimen/control_corner_material" />
        <solid android:color="@color/white" />
        <padding android:left="@dimen/button_padding_horizontal_material"
                 android:top="@dimen/button_padding_vertical_material"
                 android:right="@dimen/button_padding_horizontal_material"
                 android:bottom="@dimen/button_padding_vertical_material" />
    </shape>
</inset>
```

button_inset_horizontal_material	4dp

button_inset_vertical_material		6dp

control_corner_material		2dp

button_padding_horizontal_material		8dp

button_padding_vertical_material		4dp

colorControlHighlight引用ripple_material_dark(#4dffffff)或ripple_material_light(#1f000000)

> 可通过在自定义的Theme中添加colorControlHighlight来修改ripple的颜色



## ripple波纹效果

[RippleDrawable](https://developer.android.com/reference/android/graphics/drawable/RippleDrawable.html)

[iOS Layer 之 Mask](http://www.cocoachina.com/ios/20161014/17747.html)

当state变化时，rippleDrawable会有波纹效果，锚点可以使用setHotspot(float, float)设置。RippleDrawable可能有多个字图层，包括一个特殊的遮罩层（mask layer，该层不会直接绘制到屏幕上，用于定义父图层的部分可见区域，其颜色不会影响显示），可以直接在xml中将id设为mask作为遮罩层，借用参考文章中的一幅图

![mask_layer](https://ws3.sinaimg.cn/large/006tKfTcgy1fin909nv0zj30xc0d80tg.jpg)

代码：

除了设置ripple的color，还可以使用radius来设置ripple的半径

```xml
<ripple xmlns:android="http://schemas.android.com/apk/res/android" 
        android:color="#0f0" > 
  <item android:id="@android:id/mask" 
        android:drawable="@drawable/ic_launcher"/> 
  <item android:drawable="@android:color/darker_gray"/> 
</ripple>
```

长按button后显示的效果：

![mask_layer_ic](https://ws2.sinaimg.cn/large/006tKfTcgy1fin90940sfj305j03vjr7.jpg)

也可以在运行时使用setId(..., android.R.id.mask)设置mask层，或者用setDrawableByLayerId(android.R.id.mask, ...)来替换现有的mask layer

每一个item都相当于一个图层，如果不设置任何图层，则ripple drawable根据父视图来绘制

```xml
<ripple xmlns:android="http://schemas.android.com/apk/res/android" 
        android:color="#0f0" > 
</ripple>
```

长按后效果：

![ripple_no_child_layer](https://ws1.sinaimg.cn/large/006tKfTcgy1fin90bcvv6j304f042jr8.jpg)

设置一个纯色的mask layer

```xml
<ripple xmlns:android="http://schemas.android.com/apk/res/android"
    android:color="#0f0" >
    <item
        android:id="@android:id/mask"
        android:drawable="@android:color/darker_gray"/>
</ripple>
```

长按效果（没有按下时颜色是纯透明的，这也说明，mask layer的颜色并不影响显示效果）

![ripple_mask_layer](https://ws2.sinaimg.cn/large/006tKfTcgy1fin908h8khj304g02k743.jpg)



从上面也能看出，item的默认shape是rectangle

> 如果设置了mask layer，ripple会用该图层来进行遮罩(masked)；如果没有设置mask layer，ripple将根据子图层的复合情况来进行遮罩；如果既没有mask layer也没有子图层，ripple将根据第一个有效的父视图背景来绘制。

如：

整个Activity布局为（button是直接在theme中指定了buttonStyle）：

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#00f" >
    <LinearLayout 
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:background="#000"
        android:padding="8dp"
        android:layout_centerInParent="true">
        <Button 
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="aaaaa"
        android:textSize="18sp"/>
    </LinearLayout>
</RelativeLayout>
```

然后不设置button的子图层，长按后效果为：

![rippl_no_mask_layer_and_child_layer](https://ws1.sinaimg.cn/large/006tKfTcgy1fin90a8zaaj306904d744.jpg)

### ripple drawable的color属性

在ripple标签下可定义一个color的属性，该属性除了作为波纹的颜色外，还会在状态变化（focused、enabled、pressed）时起作用。

在RippleDrawable的onStateChange方法里会进行处理，最终调用了RippleBackground.java的enter(focused)方法

```java
    /**
     * Starts the enter animation.
     */
    public void enter(boolean fast) {
        cancel();

        final ObjectAnimator opacity = ObjectAnimator.ofFloat(this, "outerOpacity", 0, 1);
        opacity.setAutoCancel(true);
        opacity.setDuration(fast ? ENTER_DURATION_FAST : ENTER_DURATION);
        opacity.setInterpolator(LINEAR_INTERPOLATOR);

        mAnimOuterOpacity = opacity;

        // Enter animations always run on the UI thread, since it's unlikely
        // that anything interesting is happening until the user lifts their
        // finger.
        opacity.start();
    }
```

在这里会进行背景颜色的切换动画，outerOpacity为不透明度。在Material中该颜色值可通过android:colorControlHighlight属性设置。示例：将一个button的设置focusable和focusableInTouchMode为true，然后在显示时requestFocus，此时的颜色与长按另一个未获得焦点的button得到的波纹颜色是一模一样的。


