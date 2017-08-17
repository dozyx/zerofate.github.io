---
title: MediaController 分析
tags:
  - android
  - media
date: 2017-03-31 15:37:22
categories: 笔记
---



> + MediaController父类是FrameLayout
> + 尽管MediaController可以在xml中定义，但在4.4上试验会crash，因为mDecor导致了NullPointerException。查看源码貌似是必然的。。。可能正因为此，文档说明中才写明了“The way to use this class is to instantiate it programatically.”
> + MediaController以悬浮窗形式存在
> + 进度条最大值固定为1000
> + MediaController没有直接设置自定义布局的方法，但其加载视图方法 `makeControllerView()` 是protected的，所以可以继承MediaController来进行修改。。。这是一开始以为的，后面注意到无法重写，才发现该方法为 @hide ，且文档中说明 “This doesn't work as advertised” ，所以==MediaController是无法更改布局的== 
> + MediaController 在使用上具有很多的限制，但更应该看到的是其将播放逻辑与视图分离的方式
> + MediaController 并不一定要与VideoView搭配使用，真正与MediaController联系起来的地方是setAnchorView。如果使用的是VideoView，则锚点视图为VideoView的父类。MediaController实例会被放置在锚点视图的底部并水平居中。



​	MediaController的父类是FrameLayout，用于控制MediaPlayer。其功能有：

+ 播放/暂停
+ 回退rewind（5秒）
+ 快进（15秒）
+ 进度滑动条



​	MediaController会创建一套默认的控件，然后将其作为浮动窗口置于应用之上。特别地，这些控件将悬浮在使用`setAnchorView()`指定的视图之上。窗口会在空闲 3 秒后消失然后当用户点击锚视图时重新浮现。



## 布局

MediaController默认布局：

**media_controller.xml**

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:background="#CC000000"
    android:orientation="vertical">
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:gravity="center"
        android:paddingTop="4dip"
        android:layoutDirection="ltr"
        android:orientation="horizontal">
        <ImageButton android:id="@+id/prev" style="@android:style/MediaButton.Previous" />
        <ImageButton android:id="@+id/rew" style="@android:style/MediaButton.Rew" />
        <ImageButton android:id="@+id/pause" style="@android:style/MediaButton.Play" />
        <ImageButton android:id="@+id/ffwd" style="@android:style/MediaButton.Ffwd" />
        <ImageButton android:id="@+id/next" style="@android:style/MediaButton.Next" />
    </LinearLayout>
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">
        <TextView android:id="@+id/time_current"
            android:textSize="14sp"
            android:textStyle="bold"
            android:paddingTop="4dip"
            android:paddingStart="4dip"
            android:layout_gravity="center_horizontal"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:paddingEnd="4dip"
            android:textColor="@color/dim_foreground_dark" />
        <SeekBar
            android:id="@+id/mediacontroller_progress"
            style="?android:attr/progressBarStyleHorizontal"
            android:layout_width="0dip"
            android:layout_weight="1"
            android:layout_height="32dip"
            android:layout_alignParentStart="true"
            android:layout_alignParentEnd="true" />
        <TextView android:id="@+id/time"
            android:textSize="14sp"
            android:textStyle="bold"
            android:paddingTop="4dip"
            android:paddingEnd="4dip"
            android:layout_gravity="center_horizontal"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:paddingStart="4dip"
            android:textColor="@color/dim_foreground_dark" />
    </LinearLayout>
</LinearLayout>
```



### 布局隐藏规则

一些button将根据以下规则进行显示和隐藏：

- “previous” 和 “next” 的button将隐藏，直到 `setPrevNextListener()` 被调用
- 如果`setPrevNextListener` 传入的是 null 的listener，“previous” 和 “next” 的button将可见但disabled
- “rewind” 和 “fastforward” 按钮将显示，除非 MediaController(Context, boolean) 构造方法中的 boolean 为 false




### 显示位置

​	根据锚点视图来确定MediaController的坐标位置

```java
    private OnLayoutChangeListener mLayoutChangeListener =
            new OnLayoutChangeListener() {
        public void onLayoutChange(View v, int left, int top, int right,
                int bottom, int oldLeft, int oldTop, int oldRight,
                int oldBottom) {
            updateFloatingWindowLayout();
            if (mShowing) {
                mWindowManager.updateViewLayout(mDecor, mDecorLayoutParams);
            }
        }
    };
    private void updateFloatingWindowLayout() {
        int [] anchorPos = new int[2];
        mAnchor.getLocationOnScreen(anchorPos);

        // we need to know the size of the controller so we can properly position it
        // within its space
        mDecor.measure(MeasureSpec.makeMeasureSpec(mAnchor.getWidth(), MeasureSpec.AT_MOST),
                MeasureSpec.makeMeasureSpec(mAnchor.getHeight(), MeasureSpec.AT_MOST));

        WindowManager.LayoutParams p = mDecorLayoutParams;
        p.width = mAnchor.getWidth();
        p.x = anchorPos[0] + (mAnchor.getWidth() - p.width) / 2;//MediaControl水平居中
        p.y = anchorPos[1] + mAnchor.getHeight() - mDecor.getMeasuredHeight();//MediaController与锚点视图的底部对齐
    }
```




## 构造方法

MediaController有三个构造方法：

```java
    public MediaController(Context context, AttributeSet attrs) {//MediaController定义在xml中
        super(context, attrs);
        mRoot = this;
        mContext = context;
        mUseFastForward = true;
        mFromXml = true;//表示MediaController直接定义在xml中
    }
    public MediaController(Context context, boolean useFastForward) {//在代码中加载布局
        super(context);
        mContext = context;
        mUseFastForward = useFastForward;
        initFloatingWindowLayout();//初始化悬浮窗布局
        initFloatingWindow();//初始化悬浮窗
    }
    public MediaController(Context context) {//启动快进功能
        this(context, true);
    }
```

## setAnchorView

​	==为控件视图设置作为锚点的视图。==可以为 VideoView ，也可以是Activity的主视图。==当 VideoView 调用此方法时，将使用 VideoView 的父视图作为锚点。==

源码：

```java
    public void setAnchorView(View view) {
        if (mAnchor != null) {
            mAnchor.removeOnLayoutChangeListener(mLayoutChangeListener);
        }
        mAnchor = view;//更新锚点视图
        if (mAnchor != null) {
            mAnchor.addOnLayoutChangeListener(mLayoutChangeListener);
        }
        FrameLayout.LayoutParams frameParams = new FrameLayout.LayoutParams(
                ViewGroup.LayoutParams.MATCH_PARENT,
                ViewGroup.LayoutParams.MATCH_PARENT
        );
        removeAllViews();//移除旧的视图
        View v = makeControllerView();
        addView(v, frameParams);
    }
    protected View makeControllerView() {
        LayoutInflater inflate = (LayoutInflater) mContext.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
        mRoot = inflate.inflate(com.android.internal.R.layout.media_controller, null);
        initControllerView(mRoot);
        return mRoot;
    }
```



## MediaPlayerControl接口

​	该接口用于MediaController控制MediaPlayer实例，通过 `setMediaPlayer(MediaPlayerControl player)` 设置。

```java
    public interface MediaPlayerControl {
        void    start();
        void    pause();
        int     getDuration();
        int     getCurrentPosition();
        void    seekTo(int pos);
        boolean isPlaying();
        int     getBufferPercentage();
        boolean canPause();
        boolean canSeekBackward();
        boolean canSeekForward();
      
        int     getAudioSessionId();
    }
```

> 上一曲、下一曲通过额外的接口控制



## 进度条

​	进度条的最大值固定为1000，然后根据进度来确定位置，这样在滑动的单位距离都是一致的，与视频长度无关。

```java
    private int setProgress() {
        if (mPlayer == null || mDragging) {
            return 0;
        }
        int position = mPlayer.getCurrentPosition();
        int duration = mPlayer.getDuration();
        if (mProgress != null) {
            if (duration > 0) {
                // use long to avoid overflow
                long pos = 1000L * position / duration;
                mProgress.setProgress( (int) pos);
            }
            int percent = mPlayer.getBufferPercentage();
            mProgress.setSecondaryProgress(percent * 10);
        }

        if (mEndTime != null)
            mEndTime.setText(stringForTime(duration));
        if (mCurrentTime != null)
            mCurrentTime.setText(stringForTime(position));

        return position;
    }
```

​	除第一次外的每次的进度条刷新都发生在整秒

```java
    private Handler mHandler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            int pos;
            switch (msg.what) {
                case FADE_OUT:
                    hide();
                    break;
                case SHOW_PROGRESS:
                    pos = setProgress();
                    if (!mDragging && mShowing && mPlayer.isPlaying()) {
                        msg = obtainMessage(SHOW_PROGRESS);
                        sendMessageDelayed(msg, 1000 - (pos % 1000));
                    }
                    break;
            }
        }
    };
```













