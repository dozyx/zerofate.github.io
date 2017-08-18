---
title: IntentService 源码分析
tags:
  - android
date: 2017-07-04 15:37:22
categories: 笔记
---

[ExtendingIntentService](https://developer.android.com/guide/components/services.html#ExtendingIntentService)

​	IntentService 的父类为 Service，它与 Service 的区别在于内置了一个工作线程用于耗时操作。IntentService 的基本用法就是继承 IntentService，然后在 onHandleIntent(Intent) 中执行耗时操作，但由于 IntentService 只启动了一个工作线程，所有的请求只能按顺序执行。

​	IntentService 做了以下处理：

+ 创建默认的工作线程，用于在应用的主线程外执行传递给 onStartCommand() 的所有 Intent
+ 创建工作队列，用于将 Intent 逐一传递给 onHandleIntent() 实现
+ 在处理完所有启动请求后停止服务，因此不必主动调用 stopSelf()
+ 提供 onBind() 的默认实现(返回null)
+ 提供 onStartCommand() 的默认实现，可将 Intent 依次发送给工作队列和 onHandleIntent() 实现



### 源码分析

​	IntentService 只有一个构造函数

`public IntentService(String name)`

​	name 用于工作线程的名称，在继承 IntentService 实现我们自己的 Service 时，可以在我们实现的空构造方法中调用super(name) 进行设置。

​	在 onCreate() 中，IntentService 创建了一个自带 Looper 的 mServiceHandler，然后在 onStartCommand() 中将 Intent 以 Message 的方式转给 mServiceHandler。mServiceHandler 再将消息转到 onHandleIntent() 中处理。

​	需要注意的是，因为 Intent 的处理是从 onStartCommand() 方法开始的，如果只以 bindService() 方式启动服务， onStartCommand() 将不会回调，这样将导致 onHandleIntent() 无法回调。所以，一般启动IntentService，只需要调用startService()。

```java
...
    private final class ServiceHandler extends Handler {
        public ServiceHandler(Looper looper) {
            super(looper);
        }

        @Override
        public void handleMessage(Message msg) {
            onHandleIntent((Intent)msg.obj);
            stopSelf(msg.arg1);
        }
    }
    ...
    @Override
    public void onCreate() {
        // TODO: It would be nice to have an option to hold a partial wakelock
        // during processing, and to have a static startService(Context, Intent)
        // method that would launch the service & hand off a wakelock.

        super.onCreate();
        HandlerThread thread = new HandlerThread("IntentService[" + mName + "]");
        thread.start();

        mServiceLooper = thread.getLooper();
        mServiceHandler = new ServiceHandler(mServiceLooper);
    }
    
    @Override
    public void onStart(Intent intent, int startId) {
        Message msg = mServiceHandler.obtainMessage();
        msg.arg1 = startId;
        msg.obj = intent;
        mServiceHandler.sendMessage(msg);
    }
    
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        onStart(intent, startId);
        return mRedelivery ? START_REDELIVER_INTENT : START_NOT_STICKY;
    }
    ...
```