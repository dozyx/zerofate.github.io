---
title: API Guides - 属性动画
tags:
  - android
  - 动画
date: 2017-03-29 15:37:22
categories: 笔记
---

[Property Animation API Guides](https://developer.android.com/guide/topics/graphics/prop-animation.html)

​	属性动画系统是一个强大的框架(framework)，它几乎可以使任何东西都具有动画。属性动画可以在一定的时间内改变一个属性（对象的一个字段）。

​	属性动画可以指定动画的以下特征：

+ 持续时间：默认300ms
+ 时间插值器：计算随时间变化的属性的值
+ 重复次数和反应（behavior）：指定动画是否重复及重复次数，还可以指定动画是否反向（reverse）执行
+ 动画集（Animator sets）：将动画组合在一起，可以一起播放或者顺序播放，也可以在指定延时后播放
+ 帧刷新延时（Frame refresh delay）：刷新动画帧的时间。默认是每10ms刷新，但app的帧刷新速度最终取决于系统当前的忙碌状态以及系统基于底层计时器执行服务的速度。




## 属性动画是如何工作的

线性动画：

变化属性为X属性，持续时间40ms，变化距离为40像素点。该动画使用的是线性插值器。

![animation-linear](/Users/zero/OneDrive/markdown\photo\动画\animation-linear.png)

非线性动画：

例子中为动画起始加速，最后减速停止

![animation-nonlinear](/Users/zero/OneDrive/markdown\photo\动画\animation-nonlinear.png)

### 属性动画的组成部分

![valueanimator](/Users/zero/OneDrive/markdown\photo\动画\valueanimator.png)

+ ValueAnimator对象记录了动画的时间相关信息，如持续时间和动画过程中的属性值。
+ ValueAnimator封装了一个动画插值器TimeInterpolator和一个确定如何计算动画属性值的TypeEvaluator。如上面的非线性动画使用的是AccelerateDecelerateInterpolator和IntEvaluator。
+ 给定动画开始属性值和结束属性值后，调用start()即可启动动画。
+ 在整个动画期间，ValueAnimator会基于动画持续时间及当前运行时间计算一个0到1的运行比例因子（*elapsed fraction*）。当ValueAnimator计算出一个运行比例因子后，它将调用设置好的TimeInterpolator来计算一个插入比例因子（*interpolated fraction*）。一个插值比例因子对应了一个运行比例因子。在上面的线性动画中，插值比例因子与运行比例因子相同。
+ 插值比例因子在运算时，ValueAnimator调用对应的TypeEvaluator来计算动画中的属性值（基于插值比例因子、开始值、结束值）。



## Property Animation与View Animation的区别

+ view animation系统只提供了View对象的动画支持
+ view animation只能使一个View对象的某几个方面具有动画，如View的缩放和旋转，但无法动画化背景颜色等
+ view animation系统只能在View被绘制时修改，而不能改变实际的View。如，动画化移动一个button，虽然button的位置被正确的绘制，但实际button点击事件的位置却没有改变
+ 使用property animation系统，将完全没有上面的限制，我们可以动画化任何属性（View或非View对象），并且对象本身将做出实际性的改动。
+ property animation系统启动动画的方式更为强健
+ 高级地，我们给想要动画化的属性如颜色、位置、大小分配动画，并且可以定义动画插值器，还可以同时执行多个动画



​	尽管view animation系统具有很多限制，但它更易于配置，并且所需要的代码更少。



## API概览

​	property animation系统的API基本都在 android.animation 中。

表一：Animator

| 类              | 描述                                       |
| -------------- | ---------------------------------------- |
| ValueAnimator  | 属性动画主要的计时引擎，可以计算动画的属性的值。==属性动画可以分为两部分：计算动画的值和将这些值设置给对象和属性。ValueAnimator并没有实现第二部分==，所以我们必须通过ValueAnimator 监听计算的值的变化然后改变执行动画的对象。 |
| ObjectAnimator | ValueAnimator的子类，可以设置一个目标对象和对象属性来执行动画。大部分情况下使用的都是ObjectAnimator，但因为ObjectAnimator的一些局限，比如要求目标对象具有特定的访问函数（get方法），可能会在某些时候直接使用ValueAnimator。 |
| AnimatorSet    | 提供一种机制使动画成组执行。                           |

​	

​	Evaluators 告诉属性动画系统如何计算给定属性的值。

表二：Evaluators 

| 类/接口           | 描述                                       |
| -------------- | ---------------------------------------- |
| IntEvaluator   | 计算int属性值得默认evaluator                     |
| FloatEvaluator | 计算float属性值得默认evaluator                   |
| ArgbEvaluator  | 计算color属性的默认evaluator                    |
| TypeEvaluator  | 创建自定义evaluator的接口，如果准备执行动画的对象属性不是int、float或color，则必须实现TypeEvaluator接口来指定如何计算对象属性的动画值。 |

​	

​	一个时间插值器定义了如何根据时间来计算动画中特定的值。

表三：interpolators

| 类/接口                             | 描述                        |
| -------------------------------- | ------------------------- |
| AccelerateDecelerateInterpolator | 缓慢开始和结束，但中间加速             |
| AccelerateInterpolator           | 开始缓慢，然后加速                 |
| AnticipateInterpolator           | 开始前进，然后后退                 |
| AnticipateOvershootInterpolator  | 开始向后，然后向前，接着超过目标值，最后回到终止值 |
| BounceInterpolator               | 结束时弹跳                     |
| CycleInterpolator                | 在指定周期内重复                  |
| DecelerateInterpolator           | 快速开始，然后减速                 |
| LinearInterpolator               | 变化率是一个常量                  |
| OvershootInterpolator            | 向前行进，然后超出后回退              |
| TimeInterpolator                 | 接口，用于实现自定义的interpolator   |



## 使用ValueAnimator动画化	

例子：

```java
ValueAnimator animation = ValueAnimator.ofFloat(0f, 1f);
animation.setDuration(1000);
animation.start();
```

> 这个代码段不会对对象产生真正的效果，因为ValueAnimator没有直接作用于对象或属性。使用此代码的目的可能在于根据这些计算的值来修改对象。我们可以通过定义ValueAnimator中的监听器来处理动画期间的事件，如帧的更新，实现监听器后，可以通过getAnimatedValue()来为特定帧刷新获得计算的值。

也可以自定义动画作用的类型:

```java
ValueAnimator animation = ValueAnimator.ofObject(new MyTypeEvaluator(), startPropertyValue, endPropertyValue);
animation.setDuration(1000);
animation.start();
```



## 使用ObjectAnimator动画化

代码片段：

```java
//指定了对象和对象属性
ObjectAnimator anim = ObjectAnimator.ofFloat(foo, "alpha", 0f, 1f);
anim.setDuration(1000);
anim.start();
```

### 使用

为保证ObjectAnimator正确的更新属性，需要遵守以下内容：

+ 对象属性必须有一个`set<propertyName>()`形式的setter函数。如果不存在setter方法，这里有三个选择：
  + 在类中添加setter方法
  + 使用wrapper类，让wrapper类的有效setter方法来接收值，然后转发给原始对象
  + 使用ValueAnimator替换
+ 如果ObjectAnimator的工厂方法的`value...`参数只有一个值，则把该值作为动画的结束值。这要求对象属性必须有一个`get<propertyName>()`形式的getter方法，这样动画才能获得开始值。
+ 动画的属性的setter和getter方法必须对相同类型进行操作
+ 取决于动画的对象或属性，可能需要对View调用`invalidate()`方法来强制屏幕在动画值改变时进行重绘，这些在`onAnimationUpdate()`回调中进行。



## 使用AnimatorSet来设计多个动画

代码片段：

```java
AnimatorSet bouncer = new AnimatorSet();
bouncer.play(bounceAnim).before(squashAnim1);
bouncer.play(squashAnim1).with(squashAnim2);
bouncer.play(squashAnim1).with(stretchAnim1);
bouncer.play(squashAnim1).with(stretchAnim2);
bouncer.play(bounceBackAnim).after(stretchAnim2);
ValueAnimator fadeAnim = ObjectAnimator.ofFloat(newBall, "alpha", 1f, 0f);
fadeAnim.setDuration(250);
AnimatorSet animatorSet = new AnimatorSet();
animatorSet.play(bouncer).before(fadeAnim);
animatorSet.start();
```

1. 播放 `bounceAnim`
2. 同时播放`squashAnim1`, `squashAnim2`, `stretchAnim1`, 和`stretchAnim2` 
3. 播放`bounceBackAnim`
4. 播放`fadeAnim`



## 动画监听器

+ Animator.AnimatorListener
  + onAnimationStart() - 动画开始
  + onAnimationEnd() - 动画结束
  + onAnimationRepeat() - 重复动画
  + onAnimationCancel() - 动画被取消，仍会回调onAnimationEnd()
+ ValueAnimator.AnimatorUpdateListener
  + onAnimationUpdate() - 每一帧动画都会调用。通过getAnimatedValue()来获取当前计算的动画值



​	如果不想实现Animator.AnimatorListener接口中的所有方法，可以继承AnimatorListenerAdapter类。如：

```java
ValueAnimator fadeAnim = ObjectAnimator.ofFloat(newBall, "alpha", 1f, 0f);
fadeAnim.setDuration(250);
fadeAnim.addListener(new AnimatorListenerAdapter() {
public void onAnimationEnd(Animator animation) {
    balls.remove(((ObjectAnimator)animation).getTarget());
}
```



## 使ViewGroup的布局变化具有动画

​	我们可以使用LayoutTransition类来使ViewGroup中的布局改变具有动画。在添加、移除或者可见性时，ViewGroup中的View可以有一个出现和消失的动画。ViewGroup中剩下的View在添加或移除View后也可以使用动画进入新的位置。

​	在一个LayoutTransition对象中通过调用 setAnimator()并传入一个Animator对象和一个LayoutTransition常量，就可以实现下面的动画：

+ APPEARING ：作用于container中要出现的item
+ CHANGE_APPEARING ：作用于container中由于新item出现而发生改变的item
+ DISAPPEARING ：作用于container中将消失的item
+ CHANGE_DISAPPEARING ：作用于container中由于item消失而发生改变的item



例子：在xml中为ViewGroup启用默认的布局过渡

```xml
<LinearLayout
    android:orientation="vertical"
    android:layout_width="wrap_content"
    android:layout_height="match_parent"
    android:id="@+id/verticalContainer"
    android:animateLayoutChanges="true" />
```



## 使用TypeEvaluator

​	如果要用一个Android系统未知的类型来进行动画，需要创建自己的evaluator。

FloatEvaluator的实现：

```java
public class FloatEvaluator implements TypeEvaluator {
    public Object evaluate(float fraction, Object startValue, Object endValue) {
        float startFloat = ((Number) startValue).floatValue();
        return startFloat + fraction * (((Number) endValue).floatValue() - startFloat);
    }
}
```

> fraction比例因子使计算动画值时不需要考虑插值器。



## 使用Interpolators

​	插值器定义了动画中的值如何根据时间进行计算。插值器通过修改表示动画已过去时间的比例因子，来符合想要实现的动画类型。

​	可以通过实现TimeInterpolator接口来创建Interpolator，以下为两个系统Interpolator的实现：

```java
//AccelerateDecelerateInterpolator
public float getInterpolation(float input) {
    return (float)(Math.cos((input + 1) * Math.PI) / 2.0f) + 0.5f;
}
```

```java
//LinearInterpolator
public float getInterpolation(float input) {
    return input;
}
```



## 指定关键帧（Keyframe）

​	Keyframe对象由一个"时间/值"对组成，它可以用来定义一个动画某一时间的某一状态。每个关键帧都可以有自己的插值器来控制前一关键帧与该关键帧之间的动画效果。

```java
Keyframe kf0 = Keyframe.ofFloat(0f, 0f);
Keyframe kf1 = Keyframe.ofFloat(.5f, 360f);
Keyframe kf2 = Keyframe.ofFloat(1f, 0f);
PropertyValuesHolder pvhRotation = PropertyValuesHolder.ofKeyframe("rotation", kf0, kf1, kf2);
ObjectAnimator rotationAnim = ObjectAnimator.ofPropertyValuesHolder(target, pvhRotation)
rotationAnim.setDuration(5000ms);
```



## 使View具有动画

​	view animation system通过改变view的绘制方式来实现变换，这些都是在view的容器中进行，因为view本身没有可控制的属性；而property animation system 可以通过改变 View 对象的实际属性来使 View 具有动画。

​	View类的以下新属性有助于使用 property animation ：

+ translationX 和translationY：控制View位置与其布局容器所确定的left和right（getLeft和getTop）间的相对位置
+ rotation，rotation和rotationY：控制2D旋转（rotation属性）和3D旋转（围绕轴心点 pivot point）
+ scaleX 和 scaleY：控制View围绕其轴心点进行2D缩放
+ pivotX 和pivotY：控制轴心点的位置，旋转和缩放都是围绕它进行变换。默认的轴心点位置位于对象的中心。
+ x 和 y：简单的工具属性，用来描述View在容器中的最终位置，与 left/top 和translationX/translationY的总和一致
+ alpha：表示View的透明度。默认为1不透明。



​	示例代码：

```java
ObjectAnimator.ofFloat(myView, "rotation", 0f, 360f);
```



### 使用ViewPropertyAnimator

​	ViewPropertyAnimator提供了使几个属性同时具有动画的一种简单方式，只需要使用一个Animator 对象。它与 ObjectAnimator 很像，因为它修改的是view属性的实际值，但在一次动画化多个属性时有更好的效率。

​	以下代码为同时动画化 x 和 y 属性的几种方式：

**多个 ObjectAnimator 对象**

```java
ObjectAnimator animX = ObjectAnimator.ofFloat(myView, "x", 50f);
ObjectAnimator animY = ObjectAnimator.ofFloat(myView, "y", 100f);
AnimatorSet animSetXY = new AnimatorSet();
animSetXY.playTogether(animX, animY);
animSetXY.start();
```

**单个 ObjectAnimator 对象**

```java
PropertyValuesHolder pvhX = PropertyValuesHolder.ofFloat("x", 50f);
PropertyValuesHolder pvhY = PropertyValuesHolder.ofFloat("y", 100f);
ObjectAnimator.ofPropertyValuesHolder(myView, pvhX, pvyY).start();
```

**ViewPropertyAnimator**

```java
myView.animate().x(50f).y(100f);
```



## 在 XML 中声明动画

​	为区分使用传统 view animation framework API 和使用新property animation API 的动画文件，从 Android 3.1 开始，需要把属性动画的 XML 文件放置到 res/animator/ 目录。

​	

以下的属性动画类支持xml标签：

+ ValueAnimator - \<animator\>
+ ObjectAnimator - \<objectAnimator\>
+ AnimatorSet - \<set\>



代码片段：

```java
<set android:ordering="sequentially">
    <set>
        <objectAnimator
            android:propertyName="x"
            android:duration="500"
            android:valueTo="400"
            android:valueType="intType"/>
        <objectAnimator
            android:propertyName="y"
            android:duration="500"
            android:valueTo="300"
            android:valueType="intType"/>
    </set>
    <objectAnimator
        android:propertyName="alpha"
        android:duration="500"
        android:valueTo="1f"/>
</set>
```

​	为运行以上动画，需要在代码中将XML资源加载进 AnimatorSet 对象，然后在启动动画集之前为所有动画设置target对象。使用代码：

```java
AnimatorSet set = (AnimatorSet) AnimatorInflater.loadAnimator(myContext,
    R.anim.property_animator);
set.setTarget(myObject);
set.start();
```

也可以在 XML 中声明一个 ValueAnimator，如：

```xml
<animator xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="1000"
    android:valueType="floatType"
    android:valueFrom="0f"
    android:valueTo="-100f" />
```

然后，使用：

```xml
ValueAnimator xmlAnimator = (ValueAnimator) AnimatorInflater.loadAnimator(this,
        R.animator.animator);
xmlAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
    @Override
    public void onAnimationUpdate(ValueAnimator updatedAnimation) {
        float animatedValue = (float)updatedAnimation.getAnimatedValue();
        textView.setTranslationX(animatedValue);
    }
});

xmlAnimator.start();
```








