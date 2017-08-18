---
title: SimpleAdapter 分析使用
tags:
  -  android
date: 2016-12-28 15:37:22
categories: 笔记
---

[SimpleAdapter](https://developer.android.com/reference/android/widget/SimpleAdapter.html)

public class SimpleAdapter 
extends BaseAdapter implements Filterable, ThemedSpinnerAdapter

java.lang.Object
   ↳	android.widget.BaseAdapter
 	   ↳	android.widget.SimpleAdapter

------

​	SimpleAdapter 可以将数据直接映射到定义在 XML 中的 view 中，其默认支持的 View 只用三种：Checkable、TextView、ImageView。如果需要添加其他类型控件的支持，则可以通过 setViewBinder() 方法设置一个 ViewBinder 对象。

​	将数据绑定到 view 的时候，分为两个阶段：首先，如果 SimpleAdapter.ViewBinder 可用，将调用 setViewValue(android.view.View, Object, String) 方法，其结果返回 true 则表示发生绑定，否则的话，再对其余的 view 进行以下判断：

+ view 是否继承于 Checkable (如 CheckBox)，预期绑定的值为 boolean 值
+ TextView，预期绑定值为 string，将调用  setViewText(TextView, String)
+ ImageView，预期绑定值为资源 id 或者为 string，将调用  setViewImage(ImageView, int) 或者 setViewImage(ImageView, String)



### 功能

​	SimpleAdapter 实现了以下功能：

+ 基本的 Adapter 方法：getCount()、getItem()、getItemId()
+ getView()：创建 item 的视图
+ ThemedSpinnerAdapter 接口：用于 Spinner
+ Filterable 接口：实现了继承 Filter 的SimpleFilter类，可根据前缀过滤数据
+ ViewBinder：可通过 setViewBinder 设置一个 SimpleAdapter.ViewBinder 对象，在 bind view 时将先使用 ViewBinder 的 setViewValue() 方法来实现自定义的绑定，通过 setViewValue() 方法可以实现对默认的三种控件以外的控件的数据绑定
+ 绑定数据方法：setViewText()、setViewImage()。TextView 和 ImageView 发生绑定时将使用这两个方法，但只有 ViewBinder 不存在或者 ViewBinder 中没进行绑定才会调用。因为它们都是 public 方法，所以如果需要在绑定时进行一些调整可以进行重写，但需要注意的是别忘了调用 super 的实现或者自己添加绑定(自己重新添加绑定一般没必要)。



### 源码分析

#### 创建

​	SimpleAdapter 只有一个构造方法。

`SimpleAdapter(Context context, List<? extends Map<String, ?>> data, int resource, String[] from, int[] to)`

+ data：List 类型，其中的每一个 map 对应于一个行的数据
+ resource：item 的布局文件
+ from：map 中的数据源的键
+ to：与 from 中数据相对应的 view 的 id



#### 视图绑定

​	与其他 Adapter 一样，在 getView() 中创建视图

```java
    public View getView(int position, View convertView, ViewGroup parent) {
        return createViewFromResource(mInflater, position, convertView, parent, mResource);
    }
    private View createViewFromResource(LayoutInflater inflater, int position, View convertView,
            ViewGroup parent, int resource) {
        View v;
        if (convertView == null) {
            v = inflater.inflate(resource, parent, false);
        } else {
            v = convertView;
        }

        bindView(position, v);

        return v;
    }
```

​	实际的绑定发生在 bindView() 中

```java
private void bindView(int position, View view) {
        final Map dataSet = mData.get(position);
        if (dataSet == null) {
            return;
        }

        final ViewBinder binder = mViewBinder;
        final String[] from = mFrom;
        final int[] to = mTo;
        final int count = to.length;

        for (int i = 0; i < count; i++) {
            final View v = view.findViewById(to[i]);
            if (v != null) {
                final Object data = dataSet.get(from[i]);
                String text = data == null ? "" : data.toString();
                if (text == null) {
                    text = "";
                }

                boolean bound = false;
                if (binder != null) {
                    bound = binder.setViewValue(v, data, text);
                }
              	//先使用 ViewBinder 进行绑定，如果绑定发生，将不会继续

                if (!bound) {
                    if (v instanceof Checkable) {//Checkable 绑定
                        if (data instanceof Boolean) {
                            ((Checkable) v).setChecked((Boolean) data);
                        } else if (v instanceof TextView) {
                            // Note: keep the instanceof TextView check at the bottom of these
                            // ifs since a lot of views are TextViews (e.g. CheckBoxes).
                            setViewText((TextView) v, text);
                        } else {
                            throw new IllegalStateException(v.getClass().getName() +
                                    " should be bound to a Boolean, not a " +
                                    (data == null ? "<unknown type>" : data.getClass()));
                        }
                    } else if (v instanceof TextView) {//TextView 绑定
                        // Note: keep the instanceof TextView check at the bottom of these
                        // ifs since a lot of views are TextViews (e.g. CheckBoxes).
                        setViewText((TextView) v, text);
                    } else if (v instanceof ImageView) {//ImageView 绑定
                        if (data instanceof Integer) {
                            setViewImage((ImageView) v, (Integer) data);                            
                        } else {
                            setViewImage((ImageView) v, text);
                        }
                    } else {
                        throw new IllegalStateException(v.getClass().getName() + " is not a " +
                                " view that can be bounds by this SimpleAdapter");
                    }
                }
            }
        }
    }
```



### 总结

​	SimpleAdapter 只有一个构造方法，可以在构造时就添加好需要数据和对应的视图。SimpleAdapter 默认只支持 **Checkable、TextView、ImageView** 的绑定，如果需要进行其他类型控件的绑定，可以通过自定义一个 ViewBinder 对象来实现。在 ViewBinder 中绑定完成的视图，bindView() 不会继续进行绑定，即使它们是默认的三种控件之一(不过个人认为一般也不需要在 ViewBinder 中处理这三种控件，大不了就在 setViewText() 和 setViewImage() 中处理)。

​	在使用 SimpleAdapter 时需要注意以下问题：

+  SimpleAdapter 没有实现 ViewHolder缓存，每一次绑定都会重新调用 findViewById()
+  SimpleAdapter 没有数据操作的方法