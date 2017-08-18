---
title: RecyclerView 基础
tags:
  - android
date: 2016-11-03 15:37:22
categories: 笔记
---

## 开发者网站

[RecyclerView](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.html)

### 继承关系

java.lang.Object
   ↳	android.view.View
 	   ↳	android.view.ViewGroup
 	 	   ↳	android.support.v7.widget.RecyclerView
Known Direct Subclasses
HorizontalGridView, VerticalGridView

### 名词术语

（此处只是对原文名词解释的中文翻译，方便阅读英文时理解，如不打算看英文资料，可先略过，在遇到时再查阅）

**Adapter**

RecyclerView.Adapter的子类，responsible for providing views that represent items in a data set.

**Position**

Adapter内部数据项的位置

**Index**

在调用getChildAt(int)时使用的关联的子视图的索引。与Positon对应。

**Binding**

为显示与Adapter对应的一个位置的数据而对子视图进行准备的过程。

**Recycle(view)**

一个先前用于为adapter指定位置显示数据的视图可能被置入缓存中，以便接下来为再次显示相同类型的数据时进行重复利用。

**Scrap(view)**

废弃视图。一个子视图在布局时进入了临时分离的状态，scrap view在没有变成与RecyclerView（未修改但没有rebinding请求或者是被adapter修改的视图将被认为是dirty的）完全分离之前可能会被复用。

**Dirty(view)**

一个子视图在显示之前必须被adapter重新绑定。

### Positions in RecyclerView

在RecyclerView中有两种更方法相关的类型的position：

- layout position：一个item在最新的布局计算中的position，该position相对于LayoutManager。(e.g. getLayoutPosition(), findViewHolderForLayoutPosition(int))
- adapter position：adapter中一个item的position。该position相对于Adapter。(e.g. getAdapterPosition(), findViewHolderForAdapterPosition(int)) 

这两个position除了在调度adapter.notify*和计算更新的布局时，都是一样的。

### 内部类

**RecyclerView.Adapter**

class RecyclerView.Adapter<VH extends RecyclerView.ViewHolder>

Adapter的基类

**RecyclerView.AdapterDataObserver**

class RecyclerView.AdapterDataObserver

用于关注RecyclerView.Adapter变化的观察者基类

**RecyclerView.ChildDrawingOrderCallback**

interface RecyclerView.ChildDrawingOrderCallback

可用于更改RecyclerView children的绘制顺序的回调接口

**RecyclerView.ItemAnimator**

**RecyclerView.ItemDecoration**

**RecyclerView.LayoutManager**

**RecyclerView.LayoutParams**

**RecyclerView.OnChildAttachStateChangeListener**

**RecyclerView.OnFlingListener**

**RecyclerView.OnItemTouchListener**

**RecyclerView.OnScrollListener**

**RecyclerView.RecycledViewPool**

**RecyclerView.Recycler**

**RecyclerView.RecyclerListener**

**RecyclerView.SimpleOnItemTouchListener**

**RecyclerView.SmoothScroller**

**RecyclerView.State**

包含当前RecyclerView 状态的有用信息，如目标滚动位置或焦点视图。

**RecyclerView.ViewCacheExtension**

**RecyclerView.ViewHolder**

## 官方demo

源码路径sdk目录\extras\android\support\samples\Support7Demos\src\com\example\android\supportv7\widget

demo中涉及了四个类文件：

RecyclerViewActivity：demo使用的Activity

SimpleStringAdapter：继承RecyclerView.Adapter\<VH extends RecyclerView.ViewHolder\>

DividerItemDecoration：继承RecyclerView.ItemDecoration

Cheeses：定义了字符串数据



方法：

setHasFixedSize (boolean hasFixedSize)设置RecyclerView的大小是否固定（RecyclerView预先知道其大小不受adapter内容影响可以优化其性能）

```java
//RecyclerViewActivity.java
package com.example.recyclerviewtest;
...
public class RecyclerViewActivity extends Activity {

    private RecyclerView mRecyclerView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        final RecyclerView rv = new RecyclerView(this);
        rv.setLayoutManager(new MyLayoutManager(this));
        rv.setHasFixedSize(true);
        rv.setLayoutParams(new ViewGroup.LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT,
                ViewGroup.LayoutParams.MATCH_PARENT));
        rv.setAdapter(new SimpleStringAdapter(this, Cheeses.sCheeseStrings) {
            @Override
            public SimpleStringAdapter.ViewHolder onCreateViewHolder(ViewGroup parent,
                    int viewType) {
                final SimpleStringAdapter.ViewHolder vh = super
                        .onCreateViewHolder(parent, viewType);
                vh.itemView.setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        final int pos = vh.getAdapterPosition();
                        if (pos == RecyclerView.NO_POSITION) {
                            return;
                        }
                        if (pos + 1 < getItemCount()) {
                            swap(pos, pos + 1);
                        }
                    }
                });
                return vh;
            }
        });
        rv.addItemDecoration(new DividerItemDecoration(this, DividerItemDecoration.VERTICAL_LIST));
        setContentView(rv);
        mRecyclerView = rv;
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        super.onCreateOptionsMenu(menu);
        MenuItemCompat.setShowAsAction(menu.add("Layout"), MenuItemCompat.SHOW_AS_ACTION_IF_ROOM);
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        mRecyclerView.requestLayout();
        return super.onOptionsItemSelected(item);
    }

    private static final int SCROLL_DISTANCE = 80; // dp

    /**
     * A basic ListView-style LayoutManager.
     */
    class MyLayoutManager extends RecyclerView.LayoutManager {

        private static final String TAG = "MyLayoutManager";

        private int mFirstPosition;//显示的第一个view的位置

        private final int mScrollDistance;

        public MyLayoutManager(Context c) {
            final DisplayMetrics dm = c.getResources().getDisplayMetrics();
            mScrollDistance = (int) (SCROLL_DISTANCE * dm.density + 0.5f);//转为像素
        }

        @Override
        public void onLayoutChildren(RecyclerView.Recycler recycler, RecyclerView.State state) {
            final int parentBottom = getHeight() - getPaddingBottom();
            final View oldTopView = getChildCount() > 0 ? getChildAt(0) : null;
            int oldTop = getPaddingTop();
            if (oldTopView != null) {
                oldTop = oldTopView.getTop();
            }

            detachAndScrapAttachedViews(recycler);

            int top = oldTop;//此处为第一个子view的top
            int bottom;
            final int left = getPaddingLeft();
            final int right = getWidth() - getPaddingRight();

            final int count = state.getItemCount();
            for (int i = 0; mFirstPosition + i < count && top < parentBottom; i++, top = bottom) {//top为前一个的bottom
                View v = recycler.getViewForPosition(mFirstPosition + i);
                addView(v, i);//将getViewForPosition获得的view添加到关联的RecyclerView
                measureChildWithMargins(v, 0, 0);//测量子view大小
                bottom = top + getDecoratedMeasuredHeight(v);//getDecoratedMeasuredHeight获取给定view测量出的
                											 //高度，已加上内置ItemDecorations的额外
                layoutDecorated(v, left, top, right, bottom);//在RecyclerView内根据坐标(包含ItemDecoration)放置给定的子view
            }
        }

        @Override
        public RecyclerView.LayoutParams generateDefaultLayoutParams() {//为RecyclerView的child创建一个默认的LayoutParams对象
            return new RecyclerView.LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT,
                    ViewGroup.LayoutParams.WRAP_CONTENT);
        }

        @Override
        public boolean canScrollVertically() {
            return true;
        }

        @Override
        public int scrollVerticallyBy(int dy, RecyclerView.Recycler recycler,
                RecyclerView.State state) {//dy为滚动的距离，以像素为单位，整数表示滑动到底部。返回值为实际滚动距离
            if (getChildCount() == 0) {
                return 0;
            }

            int scrolled = 0;//计算实际滚动距离。当滑动到边界时，绝对值比dy小。
            final int left = getPaddingLeft();
            final int right = getWidth() - getPaddingRight();
            if (dy < 0) {//滑动到顶部
                while (scrolled > dy) {//scrolled与dy同符号或者为0，此处为负，绝对值小于dy，即实际滑动距离小于dy，到达边界。需要计算滚动时新插入view的位置
                    final View topView = getChildAt(0);//RecyclerView当前的第一个子view
                    final int hangingTop = Math.max(-getDecoratedTop(topView), 0);//
                    final int scrollBy = Math.min(scrolled - dy, hangingTop);//scrolled-dy得到偏差
                    scrolled -= scrollBy;
                    offsetChildrenVertical(scrollBy);//偏移所有子view
                    if (mFirstPosition > 0 && scrolled > dy) {//scrolled > dy表示
                        mFirstPosition--;
                        View v = recycler.getViewForPosition(mFirstPosition);
                        addView(v, 0);
                        measureChildWithMargins(v, 0, 0);
                        final int bottom = getDecoratedTop(topView);
                        final int top = bottom - getDecoratedMeasuredHeight(v);
                        layoutDecorated(v, left, top, right, bottom);
                    } else {
                        break;
                    }
                }
            } else if (dy > 0) {//滑动到底部
                final int parentHeight = getHeight();
                while (scrolled < dy) {
                    final View bottomView = getChildAt(getChildCount() - 1);
                    final int hangingBottom =
                            Math.max(getDecoratedBottom(bottomView) - parentHeight, 0);
                    final int scrollBy = -Math.min(dy - scrolled, hangingBottom);
                    scrolled -= scrollBy;
                    offsetChildrenVertical(scrollBy);
                    if (scrolled < dy && state.getItemCount() > mFirstPosition + getChildCount()) {
                        View v = recycler.getViewForPosition(mFirstPosition + getChildCount());
                        final int top = getDecoratedBottom(getChildAt(getChildCount() - 1));
                        addView(v);
                        measureChildWithMargins(v, 0, 0);
                        final int bottom = top + getDecoratedMeasuredHeight(v);
                        layoutDecorated(v, left, top, right, bottom);
                    } else {
                        break;
                    }
                }
            }
            recycleViewsOutOfBounds(recycler);
            return scrolled;
        }

        @Override
        public View onFocusSearchFailed(View focused, int direction,
                RecyclerView.Recycler recycler, RecyclerView.State state) {
            final int oldCount = getChildCount();

            if (oldCount == 0) {
                return null;
            }

            final int left = getPaddingLeft();
            final int right = getWidth() - getPaddingRight();

            View toFocus = null;
            int newViewsHeight = 0;
            if (direction == View.FOCUS_UP || direction == View.FOCUS_BACKWARD) {
                while (mFirstPosition > 0 && newViewsHeight < mScrollDistance) {
                    mFirstPosition--;
                    View v = recycler.getViewForPosition(mFirstPosition);
                    final int bottom = getDecoratedTop(getChildAt(0));
                    addView(v, 0);
                    measureChildWithMargins(v, 0, 0);
                    final int top = bottom - getDecoratedMeasuredHeight(v);
                    layoutDecorated(v, left, top, right, bottom);
                    if (v.isFocusable()) {
                        toFocus = v;
                        break;
                    }
                }
            }
            if (direction == View.FOCUS_DOWN || direction == View.FOCUS_FORWARD) {
                while (mFirstPosition + getChildCount() < state.getItemCount() &&
                        newViewsHeight < mScrollDistance) {
                    View v = recycler.getViewForPosition(mFirstPosition + getChildCount());
                    final int top = getDecoratedBottom(getChildAt(getChildCount() - 1));
                    addView(v);
                    measureChildWithMargins(v, 0, 0);
                    final int bottom = top + getDecoratedMeasuredHeight(v);
                    layoutDecorated(v, left, top, right, bottom);
                    if (v.isFocusable()) {
                        toFocus = v;
                        break;
                    }
                }
            }

            return toFocus;
        }

        public void recycleViewsOutOfBounds(RecyclerView.Recycler recycler) {
            final int childCount = getChildCount();
            final int parentWidth = getWidth();
            final int parentHeight = getHeight();
            boolean foundFirst = false;
            int first = 0;
            int last = 0;
            for (int i = 0; i < childCount; i++) {
                final View v = getChildAt(i);
                if (v.hasFocus() || (getDecoratedRight(v) >= 0 &&
                        getDecoratedLeft(v) <= parentWidth &&
                        getDecoratedBottom(v) >= 0 &&
                        getDecoratedTop(v) <= parentHeight)) {
                    if (!foundFirst) {
                        first = i;
                        foundFirst = true;
                    }
                    last = i;
                }
            }
            for (int i = childCount - 1; i > last; i--) {
                removeAndRecycleViewAt(i, recycler);
            }
            for (int i = first - 1; i >= 0; i--) {
                removeAndRecycleViewAt(i, recycler);
            }
            if (getChildCount() == 0) {
                mFirstPosition = 0;
            } else {
                mFirstPosition += first;
            }
        }
    }
}

```



## Training

[Creating Lists and Cards](https://developer.android.com/training/material/lists-cards.html#RVExamples)

为了使用RecyclerView控件，必须指定一个adapter和一个layout manager。我们通过继承RecyclerView.Adapter来创建adapter。layout manager用来确定RecyclerView中item view的位置和决定什么时候对用户不可见的item view进行重复利用。

RecyclerView提供了一下的内置layout manager：

- `LinearLayoutManager` 以垂直或水平的滚动列表方式显示items
- `GridLayoutManager` 以grid方式显示items
- `StaggeredGridLayoutManager` 以瀑布流（staggered grid）方式显示items

### Examples

此处代码为官网提供片段，不是完整代码，可自行补全

添加布局

```java
<!-- A RecyclerView with some commonly used attributes -->
<android.support.v7.widget.RecyclerView
    android:id="@+id/my_recycler_view"
    android:scrollbars="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"/>
```

设置layout manager并关联含有用于显示的数据的adapter

```java
public class MyActivity extends Activity {
    private RecyclerView mRecyclerView;
    private RecyclerView.Adapter mAdapter;
    private RecyclerView.LayoutManager mLayoutManager;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.my_activity);
        mRecyclerView = (RecyclerView) findViewById(R.id.my_recycler_view);

        // use this setting to improve performance if you know that changes
        // in content do not change the layout size of the RecyclerView
        mRecyclerView.setHasFixedSize(true);

        // use a linear layout manager
        mLayoutManager = new LinearLayoutManager(this);
        mRecyclerView.setLayoutManager(mLayoutManager);

        // specify an adapter (see also next example)
        mAdapter = new MyAdapter(myDataset);
        mRecyclerView.setAdapter(mAdapter);
    }
    ...
}
```

adapter用于访问数据集中的items，并为items创建视图，还可以在原始item不可视时用新的数据itme来替换一些视图的内容。ViewHolder描述了item的视图以及它在RecyclerView内部位置的元数据；`onCreateViewHolder`根据指定类型为RecyclerView构建新的ViewHolder来显示item；`onBindViewHolder`用于显示指定位置的数据，在此处更新ViewHolder.itemView的内容。

```java
public class MyAdapter extends RecyclerView.Adapter<MyAdapter.ViewHolder> {
    private String[] mDataset;

    // Provide a reference to the views for each data item
    // Complex data items may need more than one view per item, and
    // you provide access to all the views for a data item in a view holder
    public static class ViewHolder extends RecyclerView.ViewHolder {
        // each data item is just a string in this case
        public TextView mTextView;
        public ViewHolder(TextView v) {
            super(v);
            mTextView = v;
        }
    }

    // Provide a suitable constructor (depends on the kind of dataset)
    public MyAdapter(String[] myDataset) {
        mDataset = myDataset;
    }

    // Create new views (invoked by the layout manager)
    @Override
    public MyAdapter.ViewHolder onCreateViewHolder(ViewGroup parent,
                                                   int viewType) {
        // create a new view
        View v = LayoutInflater.from(parent.getContext())
                               .inflate(R.layout.my_text_view, parent, false);
        // set the view's size, margins, paddings and layout parameters
        ...
        ViewHolder vh = new ViewHolder(v);
        return vh;
    }

    // Replace the contents of a view (invoked by the layout manager)
    @Override
    public void onBindViewHolder(ViewHolder holder, int position) {
        // - get element from your dataset at this position
        // - replace the contents of the view with that element
        holder.mTextView.setText(mDataset[position]);

    }

    // Return the size of your dataset (invoked by the layout manager)
    @Override
    public int getItemCount() {
        return mDataset.length;
    }
}
```



## 实践

### 分割线

通过实现ItemDecoration抽象类，该类为RecyclerView的内部类。RecyclerView的addItemDecoration方法添加。

源码：

```java
    /**
     * An ItemDecoration allows the application to add a special drawing and layout offset
     * to specific item views from the adapter's data set. This can be useful for drawing dividers
     * between items, highlights, visual grouping boundaries and more.
     *
     * <p>All ItemDecorations are drawn in the order they were added, before the item
     * views (in {@link ItemDecoration#onDraw(Canvas, RecyclerView, RecyclerView.State) onDraw()}
     * and after the items (in {@link ItemDecoration#onDrawOver(Canvas, RecyclerView,
     * RecyclerView.State)}.</p>
     */
    public static abstract class ItemDecoration {
        /**
         * Draw any appropriate decorations into the Canvas supplied to the RecyclerView.
         * Any content drawn by this method will be drawn before the item views are drawn,
         * and will thus appear underneath the views.
         *
         * @param c Canvas to draw into
         * @param parent RecyclerView this ItemDecoration is drawing into
         * @param state The current state of RecyclerView
         */
      	//在item view被绘制之前执行，所以此方法内绘制的内容会显示在view的下面
        public void onDraw(Canvas c, RecyclerView parent, State state) {
            onDraw(c, parent);
        }
        /**
         * Draw any appropriate decorations into the Canvas supplied to the RecyclerView.
         * Any content drawn by this method will be drawn after the item views are drawn
         * and will thus appear over the views.
         *
         * @param c Canvas to draw into
         * @param parent RecyclerView this ItemDecoration is drawing into
         * @param state The current state of RecyclerView.
         */
      	//在item view被绘制之后执行，所以此方法内绘制的内容会显示在view的上面
        public void onDrawOver(Canvas c, RecyclerView parent, State state) {
            onDrawOver(c, parent);
        }
        /**
         * Retrieve any offsets for the given item. Each field of <code>outRect</code> specifies
         * the number of pixels that the item view should be inset by, similar to padding or margin.
         * The default implementation sets the bounds of outRect to 0 and returns.
         *
         * <p>
         * If this ItemDecoration does not affect the positioning of item views, it should set
         * all four fields of <code>outRect</code> (left, top, right, bottom) to zero
         * before returning.
         *
         * <p>
         * If you need to access Adapter for additional data, you can call
         * {@link RecyclerView#getChildAdapterPosition(View)} to get the adapter position of the
         * View.
         *
         * @param outRect Rect to receive the output.
         * @param view    The child view to decorate
         * @param parent  RecyclerView this ItemDecoration is decorating
         * @param state   The current state of RecyclerView.
         */
      	//通过outRect.set(left, top, right, bottom)来设置每一个Item的偏移量（类似于padding或margin）
        public void getItemOffsets(Rect outRect, View view, RecyclerView parent, State state) {
            getItemOffsets(outRect, ((LayoutParams) view.getLayoutParams()).getViewLayoutPosition(),
                    parent);
        }
    }
```

官方示例：[DividerItemDecoration](https://android.googlesource.com/platform/development/+/master/samples/Support7Demos/src/com/example/android/supportv7/widget/decorator/DividerItemDecoration.java)，RecyclerView.ItemDecoration的一个实现，可用于LinearLayoutManager，支持垂直和水平方向。

源码

```java
package com.example.android.supportv7.widget.decorator;
import android.content.Context;
import android.content.res.TypedArray;
import android.graphics.Canvas;
import android.graphics.Rect;
import android.graphics.drawable.Drawable;
import android.support.v4.view.ViewCompat;
import android.support.v7.widget.LinearLayoutManager;
import android.support.v7.widget.RecyclerView;
import android.view.View;
public class DividerItemDecoration extends RecyclerView.ItemDecoration {
    private static final int[] ATTRS = new int[]{
            android.R.attr.listDivider
    };
    public static final int HORIZONTAL_LIST = LinearLayoutManager.HORIZONTAL;
    public static final int VERTICAL_LIST = LinearLayoutManager.VERTICAL;
    private Drawable mDivider;
    private int mOrientation;
    public DividerItemDecoration(Context context, int orientation) {
        final TypedArray a = context.obtainStyledAttributes(ATTRS);
        mDivider = a.getDrawable(0);//从ATTRS数组中获取属性，此处只有一个
        a.recycle();
        setOrientation(orientation);
    }
    public void setOrientation(int orientation) {
        if (orientation != HORIZONTAL_LIST && orientation != VERTICAL_LIST) {
            throw new IllegalArgumentException("invalid orientation");
        }
        mOrientation = orientation;
    }
    @Override
    public void onDraw(Canvas c, RecyclerView parent) {
        if (mOrientation == VERTICAL_LIST) {
            drawVertical(c, parent);
        } else {
            drawHorizontal(c, parent);
        }
    }
    public void drawVertical(Canvas c, RecyclerView parent) {//列表为垂直
        final int left = parent.getPaddingLeft();
        final int right = parent.getWidth() - parent.getPaddingRight();
        final int childCount = parent.getChildCount();
        for (int i = 0; i < childCount; i++) {
            final View child = parent.getChildAt(i);
            final RecyclerView.LayoutParams params = (RecyclerView.LayoutParams) child
                    .getLayoutParams();
            final int top = child.getBottom() + params.bottomMargin +
                    Math.round(ViewCompat.getTranslationY(child));//Math.round四舍五入取整
            final int bottom = top + mDivider.getIntrinsicHeight();
            mDivider.setBounds(left, top, right, bottom);
            mDivider.draw(c);
        }
    }
    public void drawHorizontal(Canvas c, RecyclerView parent) {//列表为水平
        final int top = parent.getPaddingTop();
        final int bottom = parent.getHeight() - parent.getPaddingBottom();
        final int childCount = parent.getChildCount();
        for (int i = 0; i < childCount; i++) {
            final View child = parent.getChildAt(i);
            final RecyclerView.LayoutParams params = (RecyclerView.LayoutParams) child
                    .getLayoutParams();
            final int left = child.getRight() + params.rightMargin +
                    Math.round(ViewCompat.getTranslationX(child));
            final int right = left + mDivider.getIntrinsicHeight();
            mDivider.setBounds(left, top, right, bottom);
            mDivider.draw(c);
        }
    }
    @Override
    public void getItemOffsets(Rect outRect, int itemPosition, RecyclerView parent) {
        if (mOrientation == VERTICAL_LIST) {
            outRect.set(0, 0, 0, mDivider.getIntrinsicHeight());
        } else {
            outRect.set(0, 0, mDivider.getIntrinsicWidth(), 0);
        }
    }
}
```



### ItemAnimator动画



### SnapHelper

SnapHelper是一个继承RecyclerView.OnFlingListener接口的抽象类

```java
public static abstract class OnFlingListener {
    /**
     * Override this to handle a fling given the velocities in both x and y directions.
     * Note that this method will only be called if the associated {@link LayoutManager}
     * supports scrolling and the fling is not handled by nested scrolls first.
     */
  	//velocityX表示x轴速度，velocityY表示y轴速度
    public abstract boolean onFling(int velocityX, int velocityY);
}
```



LinearLayoutManager.java中实现了RecyclerView.SmoothScroller.ScrollVectorProvider接口

源码

```java
//这里计算的位置并不是具体的坐标值
@Override
public PointF computeScrollVectorForPosition(int targetPosition) {
     if (getChildCount() == 0) {
        return null;
    }
    final int firstChildPos = getPosition(getChildAt(0));
    final int direction = targetPosition < firstChildPos != mShouldReverseLayout ? -1 : 1;
    if (mOrientation == HORIZONTAL) {
        return new PointF(direction, 0);
    } else {
        return new PointF(0, direction);
    }
}
```



重写calculateDistanceToFinalSnap()方法来确定RecyclerView最后的targetView停止位置



在RecyclerView的fling方法中，会先调用mOnFlingListener.onFling，如果该方法把fling事件消耗掉，则fling会返回。fling方法中有一个设置最大速度的计算，感觉挺巧妙的。

`velocityX = Math.max(-mMaxFlingVelocity, Math.min(velocityX, mMaxFlingVelocity));`



