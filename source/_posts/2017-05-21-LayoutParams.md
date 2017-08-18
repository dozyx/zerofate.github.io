---
title: LayoutParams
tags:
  - android
date: 2017-05-21 15:37:22
categories: 笔记
---

## LayoutParams

​	LayoutParams ，顾名思义，表示的是布局参数，view 通过相应的 LayoutParams 来告诉它们的 parent 应该如何对它们进行布局。ViewGroup 的子类一般都会实现自己的 LayoutParams，基本的 LayoutParams 只描述了 view 所希望得到的宽和高大小，如：

+ MATCH_PARENT
+ WRAP_CONTENT
+ 确切的数值

ViewGroup.LayoutParams 源码：

```java
public static class LayoutParams {

        @SuppressWarnings({"UnusedDeclaration"})
        @Deprecated
        public static final int FILL_PARENT = -1;

        /**
         * view 希望与 parent 大小一样，需要减去 parent 的 padding。
         */
        public static final int MATCH_PARENT = -1;

        /**
         * view 希望大小满足它的内容的大小，包括 view 的 padding
         */
        public static final int WRAP_CONTENT = -2;

        /**
         * view 所希望的宽度的信息，它的值可以是 MATCH_PARENT、WRAP_CONTENT 或者确切的值
         */
        public int width;

        public int height;

        /**
         * Used to animate layouts.
         */
        public LayoutAnimationController.AnimationParameters layoutAnimationParameters;

        /**
         * 创建新的一组布局参数，它的值从指定的属性集和 context 中取出
         */
        public LayoutParams(Context c, AttributeSet attrs) {
            TypedArray a = c.obtainStyledAttributes(attrs, R.styleable.ViewGroup_Layout);
            setBaseAttributes(a,
                    R.styleable.ViewGroup_Layout_layout_width,
                    R.styleable.ViewGroup_Layout_layout_height);
            a.recycle();
        }

        /**
         * 使用指定的宽和高来构造一组新的布局参数，传入的参数为确切值时单位是像素
         */
        public LayoutParams(int width, int height) {
            this.width = width;
            this.height = height;
        }

        /**
         * 复制构造器
         */
        public LayoutParams(LayoutParams source) {
            this.width = source.width;
            this.height = source.height;
        }

        /**
         * Used internally by MarginLayoutParams.
         * @hide
         */
        LayoutParams() {
        }

        /**
         * 从提供的 attribute 中提取布局参数
         */
        protected void setBaseAttributes(TypedArray a, int widthAttr, int heightAttr) {
          	//getLayoutDimension 方法的第二个参数用于报告错误信息
            width = a.getLayoutDimension(widthAttr, "layout_width");
            height = a.getLayoutDimension(heightAttr, "layout_height");
        }

        /**
         * 根据布局方式处理布局参数，默认实现没内容，layoutDirection 可取值为：View.LAYOUT_DIRECTION_LTR和
         * View.LAYOUT_DIRECTION_RTL
         */
        public void resolveLayoutDirection(int layoutDirection) {
        }
    }
```