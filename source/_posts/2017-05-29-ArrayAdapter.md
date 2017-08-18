---
title: ArrayAdapter 类分析使用
tags:
  - android
date: 2017-05-29 15:37:22
categories: 笔记
---

java.lang.Object
   ↳	android.widget.BaseAdapter
 	   ↳	android.widget.ArrayAdapter<T>

------

​	该 adapter 用于为 AdapterView 提供视图，它将为数据对象集合中的每一个对象返回一个视图，可用于基于列表的控件如 ListView 和 Spinner。**ArrayAdapter 默认会将每个数据对象的 toString() 结果显示到一个 TextView 中。如果需要自定义数据对象所使用的 View 类型，需要重写 getView 方法。**

### 实现功能

​	ArrayAdapter 实现了以下功能：

+ 数据操作：add()、addAll()、insert()、remove()、clear() 、sort()
+ 数据改变时是否更新 UI：setNotifyOnChange()
+ 基本的 Adapter 方法：getCount()、getItem()、getItemId(...)
+ 用于 Spinner 控件的方法：setDropDownViewResource()、setDropDownViewTheme()等
+ Filterable 接口：使列表只显示指定前缀的数据
+ 其他：getPosition()



### 源码分析

#### 构造

​	ArrayAdapter 的构造函数有：

``ArrayAdapter(Context context, int resource)`

`ArrayAdapter(Context context, int resource, int textViewResourceId)`

`ArrayAdapter(Context context, int resource, T[] objects)`

`ArrayAdapter(Context context, int resource, int textViewResourceId, T[] objects)`

`ArrayAdapter(Context context, int resource, List<T> objects)`

`ArrayAdapter(Context context, int resource, int textViewResourceId, List<T> objects)`

- resource

  列表每一个 item 对应布局的 id

- textViewResourceId

  resource 表示的布局中的 TextView 的 id（ArrayAdapter 要求的布局文件必须有一个）

- objects

  数据集合，可能为数组或 List 类型，如果是数组类型，将通过 Arrays.asList 方法转化为 ArrayList。



​	除了以上构造函数外，ArrayAdapter 还提供了一个从外部资源创建 ArrayAdapter 的静态方法

`createFromResource(Context context, int textArrayResId, int textViewResId)`

#### 数据操作

​	以 add 方法为例：

```java
    public void add(@Nullable T object) {
        synchronized (mLock) {//确保线程安全
            if (mOriginalValues != null) {
                mOriginalValues.add(object);
            } else {
                mObjects.add(object);
            }
        }
        if (mNotifyOnChange) notifyDataSetChanged();
    }
```

​	mOriginalValues 为完整数据，mObjects 为实际显示在列表中的数据(可能经过过滤)



#### getView 创建视图

​	ArrayAdapter 默认的 getView 实现只是将数据显示到一个 TextView 中

```java
	@Override
    public @NonNull View getView(int position, @Nullable View convertView,
            @NonNull ViewGroup parent) {
        return createViewFromResource(mInflater, position, convertView, parent, mResource);
    }
	private @NonNull View  (@NonNull LayoutInflater inflater, int position,
        @Nullable View convertView, @NonNull ViewGroup parent, int resource) {
    final View view;
    final TextView text;

    if (convertView == null) {
        view = inflater.inflate(resource, parent, false);
    } else {
        view = convertView;
    }

    try {
        if (mFieldId == 0) {//如果没有指定 TextView 的 Id，将认为整个布局为一个 TextView
            //  If no custom field is assigned, assume the whole resource is a TextView
            text = (TextView) view;
        } else {
            //  Otherwise, find the TextView field within the layout
            text = (TextView) view.findViewById(mFieldId);

            if (text == null) {
                throw new RuntimeException("Failed to find view with ID "
                        + mContext.getResources().getResourceName(mFieldId)
                        + " in item layout");
            }
        }
    } catch (ClassCastException e) {
        Log.e("ArrayAdapter", "You must supply a resource ID for a TextView");
        throw new IllegalStateException(
                "ArrayAdapter requires the resource ID to be a TextView", e);
    }

    final T item = getItem(position);
    if (item instanceof CharSequence) {
        text.setText((CharSequence) item);
    } else {
        text.setText(item.toString());
    }

    return view;
}
```



#### Filterable 接口

​	ArrayAdapter 实现了 Filterable 接口的 getFilter() 方法，该方法返回一个 ArrayFilter 对象。ArrayFilter 可以使视图只显示指定前缀的数据。

```java
	/**
     * <p>An array filter constrains the content of the array adapter with
     * a prefix. Each item that does not start with the supplied prefix
     * is removed from the list.</p>
     */
    private class ArrayFilter extends Filter {
        @Override
        protected FilterResults performFiltering(CharSequence prefix) {
            final FilterResults results = new FilterResults();

            if (mOriginalValues == null) {
                synchronized (mLock) {
                    mOriginalValues = new ArrayList<>(mObjects);//保存完整数据
                }
            }

            if (prefix == null || prefix.length() == 0) {//无过滤
                final ArrayList<T> list;
                synchronized (mLock) {
                    list = new ArrayList<>(mOriginalValues);
                }
                results.values = list;
                results.count = list.size();
            } else {
                final String prefixString = prefix.toString().toLowerCase();

                final ArrayList<T> values;
                synchronized (mLock) {
                    values = new ArrayList<>(mOriginalValues);
                }

                final int count = values.size();
                final ArrayList<T> newValues = new ArrayList<>();

                for (int i = 0; i < count; i++) {//获取前缀满足的数据(忽略空格)
                    final T value = values.get(i);
                    final String valueText = value.toString().toLowerCase();

                    // First match against the whole, non-splitted value
                    if (valueText.startsWith(prefixString)) {
                        newValues.add(value);
                    } else {
                        final String[] words = valueText.split(" ");
                        for (String word : words) {
                            if (word.startsWith(prefixString)) {
                                newValues.add(value);
                                break;
                            }
                        }
                    }
                }

                results.values = newValues;
                results.count = newValues.size();
            }

            return results;
        }

        @Override
        protected void publishResults(CharSequence constraint, FilterResults results) {
            //noinspection unchecked
            mObjects = (List<T>) results.values;
            if (results.count > 0) {
                notifyDataSetChanged();
            } else {
                notifyDataSetInvalidated();
            }
        }
    }
```

​	mOriginalValues 用来保存一份完整数据，如果 mOriginalValues 不为 null，则在 ArrayAdapter 中进行数据操作时，基于这份数据操作。



### 总结

​	ArrayAdapter 可用于列表所有数据相同的情况，默认的实现只是将数据显示到一个 TextView 上，如果需要自定义，则需要重写 getView() 方法。

> ​	在网络上看很多文章一般建议在实现更复杂布局时直接实现 BaseAdapter，而不是继承 ArrayAdapter，这样能更灵活。可能因为缺乏经验(可以说基本没在实际项目中用过 ArrayAdapter….)，目前也不是很理解"灵活"在哪，因为 ArrayAdapter 是 BaseAdapter 的一个实现类，可以说需要继承 BaseAdapter 实现的功能都能通过继承 ArrayAdapter 来实现。 不过，如果单纯重写 ArrayAdapter 的 getView，也的确没能显示出 ArrayAdapter 有多大优势，且默认实现及没用上的其他功能都会显得多余，关于"灵活"也只能理解到这。不过，如果没什么大问题，以后还是会在可以的情况下先尝试 ArrayAdapter，毕竟不用重写 BaseAdapter 的其他几个抽象方法，而且 ArrayAdapter 与 BaseAdapter 的转化也没什么难处。(好懒。。。)