---
title: MeasureSpec
tags:
  - android
date: 2017-05-21 15:37:22
categories: 笔记
---

## MeasureSpec

​	在自定义 View 时，一般都会重写 onMeasure 方法

`protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec)`

​	widthMeasureSpec 和 heightMeasureSpec 是由 parent 给出的大小值，而这两个值正是被编码成了 MeasureSpec 的格式，所以，MeasureSpec 是我们从 widthMeasureSpec 和 heightMeasureSpec 分离提取信息的方法。

​	MeasureSpec 由两个部分组成：一个大小和一个模式。对于模式而言，有三种可能选择：

+ UNSPECIFIED

  parent 没有对 child 做出任何限制

+ EXACTLY

  parent 已经为 child 确定了一个确切的大小，child 所希望的大小只能限制在这个边界中

+ AT_MOST

  child 可以在特定大小范围内尽可能的大


​	通过 MeasureSpec.getSize(int measureSpec) 和 MeasureSpec.getMode(int measureSpec) 这两个方法，我们就可以分别得到 measureSpec 的 size 和 mode。



在 View 类中提供了 getDefaultSize 方法来返回默认大小值

```java
    public static int getDefaultSize(int size, int measureSpec) {
        int result = size;
        int specMode = MeasureSpec.getMode(measureSpec);
        int specSize = MeasureSpec.getSize(measureSpec);

        switch (specMode) {
        case MeasureSpec.UNSPECIFIED:
            result = size;
            break;
        case MeasureSpec.AT_MOST:
        case MeasureSpec.EXACTLY:
            result = specSize;
            break;
        }
        return result;
    }
```

同时，还提供了 getSuggestedMinimumWidth 和 getSuggestedMinimumHeight 来获得建议的最小值，该最小值取的是最小宽度/高度（mMinWidth/mMinHeight）和 背景（mBackgroup）最小值之间的最大值。

```java
protected int getSuggestedMinimumWidth() {
     return (mBackground == null) ? mMinWidth : max(mMinWidth, mBackground.getMinimumWidth());
}
```



