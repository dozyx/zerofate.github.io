---
title: CursorAdapter 分析使用
tags:
  - android
date: 2017-02-18 15:37:22
categories: 笔记
---

[CursorAdapter](https://developer.android.com/reference/android/widget/CursorAdapter.html)

public abstract class CursorAdapter 
extends BaseAdapter implements Filterable, ThemedSpinnerAdapter

java.lang.Object
   ↳	android.widget.BaseAdapter
 	   ↳	android.widget.CursorAdapter
  Known Direct Subclasses
ResourceCursorAdapter
  Known Indirect Subclasses
SimpleCursorAdapter

------

​	CursorAdapter 以 Cursor 作为数据，该 **Cursor 必须包含“_id”列**。此外，在使用  MergeCursor 时，如果合并的 Cursor 的 "_id" 列的值重复的话也会无法工作。

​	CursorAdapter 主要实现了以下功能：

+ 使用 Cursor 构建列表控件视图
+ 支持内容改变时自动查询(一般不推荐启用)
+ 实现 ThemedSpinnerAdapter 接口
+ 基本的 Adapter 方法：getCount() 等
+ Cursor 替换方法：changeCursor()、swapCursor()，其中 swapCursor() 将不会关闭就的 Cursor 对象，而是将其返回。
+ 内容过滤：从源码上看，应该需要自己实现查询接口，然后通过 setFilterQueryProvider() 方法设置才能使用，因为暂时没用到，只是稍微了解了一下

### 使用

​	CursorAdapter 是一个抽象类，它有两个抽象方法：

`public abstract View  newView(Context context, Cursor cursor, ViewGroup parent)`

创建一个新的视图来存放 cursor 中的数据，cursor 已移动到正确位置

`public abstract void bindView(View view, Context context, Cursor cursor)`

将视图与 cursor 中数据进行绑定，cursor 已移动到正确位置

### 源码分析

#### 初始化

​	CursorAdapter 的构造函数有两个，其中，第二个为推荐的构造函数：

`CursorAdapter(Context context, Cursor c, boolean autoRequery)`

`CursorAdapter(Context context, Cursor c, int flags)`

+ autoRequery：如果为 true ，在 cursor 发生变化时，adapter 将调用 requery() 方法进行重新查询，这样将可以一直显示最新的数据。但并**不推荐在这里使用 true**。
+ flags：用于确定 adapter 的行为，它的值可以为 FLAG_AUTO_REQUERY 和 FLAG_REGISTER_CONTENT_OBSERVER 的任意组合。
  + FLAG_AUTO_REQUERY：@deprecated，内容改变时将调用 cursor 的 requery() 方法，设置此 flag 将自动设置FLAG_REGISTER_CONTENT_OBSERVER。并不推荐此选项，因为它将导致，在 UI 线程上发生 Cursor 查询。作为替代方案，可以结合 CursorLoader 来使用 LoaderManager。
  + FLAG_REGISTER_CONTENT_OBSERVER：adapter 将对 cursor 注册一个 content observer，并在发生通知时调用 onContentChanged() 方法。谨慎使用该 flag ，因为需要注意还原 adapter 的 cursor 以避免因为注册的 observer 导致泄露。使用 CursorLoader 的 CursorAdapter 不需要该 flag。

创建初始化

```java
    void init(Context context, Cursor c, int flags) {
        if ((flags & FLAG_AUTO_REQUERY) == FLAG_AUTO_REQUERY) {
            flags |= FLAG_REGISTER_CONTENT_OBSERVER;
            mAutoRequery = true;
        } else {
            mAutoRequery = false;
        }
        boolean cursorPresent = c != null;
        mCursor = c;
        mDataValid = cursorPresent;
        mContext = context;
        mRowIDColumn = cursorPresent ? c.getColumnIndexOrThrow("_id") : -1;
        if ((flags & FLAG_REGISTER_CONTENT_OBSERVER) == FLAG_REGISTER_CONTENT_OBSERVER) {
            mChangeObserver = new ChangeObserver();
            mDataSetObserver = new MyDataSetObserver();
        } else {
            mChangeObserver = null;
            mDataSetObserver = null;
        }

        if (cursorPresent) {
          	// mChangeObserver,如果设置了自动查询，在内容改变时，将调用 mCursor 的 requery() 方法
            if (mChangeObserver != null) c.registerContentObserver(mChangeObserver);
          	// mDataSetObserver，当 mCursor 数据改变时，更新 UI
            if (mDataSetObserver != null) c.registerDataSetObserver(mDataSetObserver);
        }
    }
    protected void onContentChanged() {
        if (mAutoRequery && mCursor != null && !mCursor.isClosed()) {
            if (false) Log.v("Cursor", "Auto requerying " + mCursor + " due to update");
            mDataValid = mCursor.requery();
        }
    }
	...
    private class ChangeObserver extends ContentObserver {
        public ChangeObserver() {
            super(new Handler());
        }

        @Override
        public boolean deliverSelfNotifications() {
            return true;
        }

        @Override
        public void onChange(boolean selfChange) {
            onContentChanged();
        }
    }

    private class MyDataSetObserver extends DataSetObserver {
        @Override
        public void onChanged() {
            mDataValid = true;
            notifyDataSetChanged();
        }

        @Override
        public void onInvalidated() {
            mDataValid = false;
            notifyDataSetInvalidated();
        }
    }
```

#### 创建视图

```java
    public View getView(int position, View convertView, ViewGroup parent) {
        if (!mDataValid) {
            throw new IllegalStateException("this should only be called when the cursor is valid");
        }
        if (!mCursor.moveToPosition(position)) {
            throw new IllegalStateException("couldn't move cursor to position " + position);
        }
        View v;
        if (convertView == null) {
            v = newView(mContext, mCursor, parent);
        } else {
            v = convertView;
        }
        bindView(v, mContext, mCursor);
        return v;
    }
```