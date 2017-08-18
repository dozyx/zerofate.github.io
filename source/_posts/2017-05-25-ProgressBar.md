---
title: ProgressBar 的使用
tags:
  - android
date: 2017-05-25 15:37:22
categories: 笔记
---

## 基本

​	ProgressBar 用于显示当前进度，可以有第二进度，如播放过程中表示视频缓冲进度。进度也可以是模糊的，如一个不指示当前进度、一直旋转的圆。

​	ProgressBar 的父类为View，直接子类如 AbsSeekBar、ContentLoadingProgressBar，间接子类如AppCompatRatingBar、AppCompatSeekBar、RatingBar、SeekBar。



### style

​	系统提供了多种可选的style，如

- `Widget.ProgressBar.Horizontal`
- `Widget.ProgressBar.Small`
- `Widget.ProgressBar.Large`
- `Widget.ProgressBar.Inverse`
- `Widget.ProgressBar.Small.Inverse`
- `Widget.ProgressBar.Large.Inverse`



### 属性

\<ProgressBar> 支持以下属性

```xml
<declare-styleable name="ProgressBar">
        <!-- 最大进度值 -->
        <attr name="max" format="integer" />
        <!-- 默认进度值 -->
        <attr name="progress" format="integer" />
        <!-- 第二进度值，从 0 到 max。该进度会绘制在基础进度和背景中间。-->
        <attr name="secondaryProgress" format="integer" />
        <!-- 是否为模糊模式 -->
        <attr name="indeterminate" format="boolean" />
        <!-- Restricts to ONLY indeterminate mode (state-keeping progress mode will not work). -->
        <attr name="indeterminateOnly" format="boolean" />
        <!-- 模糊模式的 drawable -->
        <attr name="indeterminateDrawable" format="reference" />
        <!-- 进度条模式的 drawable -->
        <attr name="progressDrawable" format="reference" />
        <!-- 模糊动画持续时间 -->
        <attr name="indeterminateDuration" format="integer" min="1" />
        <!-- 模糊模式到达最大值后的行为 -->
        <attr name="indeterminateBehavior">
            <!-- 进度从 0 重新开始 -->
            <enum name="repeat" value="1" />
            <!-- 保留当前值然后回到0 -->
            <enum name="cycle" value="2" />
        </attr>
        <attr name="minWidth" format="dimension" />
        <attr name="maxWidth" />
        <attr name="minHeight" format="dimension" />
        <attr name="maxHeight" />
        <attr name="interpolator" format="reference" />
        <!-- Defines if the associated drawables need to be mirrored when in RTL mode.
             Default is false -->
        <attr name="mirrorForRtl" format="boolean" />
        <!-- Tint to apply to the progress indicator. -->
        <attr name="progressTint" format="color" />
        <!-- Blending mode used to apply the progress indicator tint. -->
        <attr name="progressTintMode">
            <!-- The tint is drawn on top of the drawable.
                 [Sa + (1 - Sa)*Da, Rc = Sc + (1 - Sa)*Dc] -->
            <enum name="src_over" value="3" />
            <!-- The tint is masked by the alpha channel of the drawable. The drawable鈥檚
                 color channels are thrown out. [Sa * Da, Sc * Da] -->
            <enum name="src_in" value="5" />
            <!-- The tint is drawn above the drawable, but with the drawable鈥檚 alpha
                 channel masking the result. [Da, Sc * Da + (1 - Sa) * Dc] -->
            <enum name="src_atop" value="9" />
            <!-- Multiplies the color and alpha channels of the drawable with those of
                 the tint. [Sa * Da, Sc * Dc] -->
            <enum name="multiply" value="14" />
            <!-- [Sa + Da - Sa * Da, Sc + Dc - Sc * Dc] -->
            <enum name="screen" value="15" />
            <!-- Combines the tint and drawable color and alpha channels, clamping the
                 result to valid color values. Saturate(S + D) -->
            <enum name="add" value="16" />
        </attr>
        <!-- Tint to apply to the progress indicator background. -->
        <attr name="progressBackgroundTint" format="color" />
        <!-- Blending mode used to apply the progress indicator background tint. -->
        <attr name="progressBackgroundTintMode">
            <!-- The tint is drawn on top of the drawable.
                 [Sa + (1 - Sa)*Da, Rc = Sc + (1 - Sa)*Dc] -->
            <enum name="src_over" value="3" />
            <!-- The tint is masked by the alpha channel of the drawable. The drawable鈥檚
                 color channels are thrown out. [Sa * Da, Sc * Da] -->
            <enum name="src_in" value="5" />
            <!-- The tint is drawn above the drawable, but with the drawable鈥檚 alpha
                 channel masking the result. [Da, Sc * Da + (1 - Sa) * Dc] -->
            <enum name="src_atop" value="9" />
            <!-- Multiplies the color and alpha channels of the drawable with those of
                 the tint. [Sa * Da, Sc * Dc] -->
            <enum name="multiply" value="14" />
            <!-- [Sa + Da - Sa * Da, Sc + Dc - Sc * Dc] -->
            <enum name="screen" value="15" />
            <!-- Combines the tint and drawable color and alpha channels, clamping the
                 result to valid color values. Saturate(S + D) -->
            <enum name="add" value="16" />
        </attr>
        <!-- Tint to apply to the secondary progress indicator. -->
        <attr name="secondaryProgressTint" format="color" />
        <!-- Blending mode used to apply the secondary progress indicator tint. -->
        <attr name="secondaryProgressTintMode">
            <!-- The tint is drawn on top of the drawable.
                 [Sa + (1 - Sa)*Da, Rc = Sc + (1 - Sa)*Dc] -->
            <enum name="src_over" value="3" />
            <!-- The tint is masked by the alpha channel of the drawable. The drawable鈥檚
                 color channels are thrown out. [Sa * Da, Sc * Da] -->
            <enum name="src_in" value="5" />
            <!-- The tint is drawn above the drawable, but with the drawable鈥檚 alpha
                 channel masking the result. [Da, Sc * Da + (1 - Sa) * Dc] -->
            <enum name="src_atop" value="9" />
            <!-- Multiplies the color and alpha channels of the drawable with those of
                 the tint. [Sa * Da, Sc * Dc] -->
            <enum name="multiply" value="14" />
            <!-- [Sa + Da - Sa * Da, Sc + Dc - Sc * Dc] -->
            <enum name="screen" value="15" />
            <!-- Combines the tint and drawable color and alpha channels, clamping the
                 result to valid color values. Saturate(S + D) -->
            <enum name="add" value="16" />
        </attr>
        <!-- Tint to apply to the indeterminate progress indicator. -->
        <attr name="indeterminateTint" format="color" />
        <!-- Blending mode used to apply the indeterminate progress indicator tint. -->
        <attr name="indeterminateTintMode">
            <!-- The tint is drawn on top of the drawable.
                 [Sa + (1 - Sa)*Da, Rc = Sc + (1 - Sa)*Dc] -->
            <enum name="src_over" value="3" />
            <!-- The tint is masked by the alpha channel of the drawable. The drawable鈥檚
                 color channels are thrown out. [Sa * Da, Sc * Da] -->
            <enum name="src_in" value="5" />
            <!-- The tint is drawn above the drawable, but with the drawable鈥檚 alpha
                 channel masking the result. [Da, Sc * Da + (1 - Sa) * Dc] -->
            <enum name="src_atop" value="9" />
            <!-- Multiplies the color and alpha channels of the drawable with those of
                 the tint. [Sa * Da, Sc * Dc] -->
            <enum name="multiply" value="14" />
            <!-- [Sa + Da - Sa * Da, Sc + Dc - Sc * Dc] -->
            <enum name="screen" value="15" />
            <!-- Combines the tint and drawable color and alpha channels, clamping the
                 result to valid color values. Saturate(S + D) -->
            <enum name="add" value="16" />
        </attr>
        <!-- Tint to apply to the background. -->
        <attr name="backgroundTint" />
        <!-- Blending mode used to apply the background tint. -->
        <attr name="backgroundTintMode" />
</declare-styleable>
```



## 修改样式

### 进度条背景

进度条背景通过 `progressDrawable` 属性设置

#### 水平进度条

 `Widget.ProgressBar.Horizontal`style 使用的drawable为 `progress_horizontal.xml`

```xml
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    
    <item android:id="@android:id/background">
        <shape>
            <corners android:radius="5dip" />
            <gradient
                    android:startColor="#ff9d9e9d"
                    android:centerColor="#ff5a5d5a"
                    android:centerY="0.75"
                    android:endColor="#ff747674"
                    android:angle="270"
            />
        </shape>
    </item>
    
    <item android:id="@android:id/secondaryProgress">
        <clip>
            <shape>
                <corners android:radius="5dip" />
                <gradient
                        android:startColor="#80ffd300"
                        android:centerColor="#80ffb600"
                        android:centerY="0.75"
                        android:endColor="#a0ffcb00"
                        android:angle="270"
                />
            </shape>
        </clip>
    </item>
    
    <item android:id="@android:id/progress">
        <clip>
            <shape>
                <corners android:radius="5dip" />
                <gradient
                        android:startColor="#ffffd300"
                        android:centerColor="#ffffb600"
                        android:centerY="0.75"
                        android:endColor="#ffffcb00"
                        android:angle="270"
                />
            </shape>
        </clip>
    </item>
    
</layer-list>
```

progress_horizontal 是一个layer-list，其中定义了三个 drawable：背景、第一进度条、第二进度条。这些 drawable 都采用了渐变的效果，以 0.75y 为中心色，270 度表示从上往下变化。