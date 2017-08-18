---
title: AsyncTask 的使用
tags:
  - android
date: 2017-02-21 15:37:22
categories: 笔记
---

[AsyncTask](https://developer.android.com/reference/android/os/AsyncTask.html)

​	AsyncTask是一个抽象类，使用该类可以实现后台执行操作，然后将结果在UI线程上展示。AsyncTask被设计来作为Thread和Handler的辅助类，并且不同于一般的线程框架。AsyncTask应使用于短时的操作（最多为几秒钟）。

>  AsyncTasks should ideally be used for short operations (a few seconds at the most.) 

​	如果需要使线程长时间运行，推荐使用`java.util.concurrent`包的API，如 [Executor](https://developer.android.com/reference/java/util/concurrent/Executor.html)、[ThreadPoolExecutor](https://developer.android.com/reference/java/util/concurrent/ThreadPoolExecutor.html) 和 [FutureTask](https://developer.android.com/reference/java/util/concurrent/FutureTask.html).



## 使用

### Developer Guides

​	AsyncTask必须被子类化才能使用，至少需要重载`(doInBackground(Params...)`方法。

​	例子（片段）：

```java
 //子类化
 private class DownloadFilesTask extends AsyncTask<URL, Integer, Long> {
     protected Long doInBackground(URL... urls) {
         int count = urls.length;
         long totalSize = 0;
         for (int i = 0; i < count; i++) {
             totalSize += Downloader.downloadFile(urls[i]);
             publishProgress((int) ((i / (float) count) * 100));
             // Escape early if cancel() is called
             if (isCancelled()) break;
         }
         return totalSize;
     }

     protected void onProgressUpdate(Integer... progress) {
         setProgressPercent(progress[0]);
     }

     protected void onPostExecute(Long result) {
         showDialog("Downloaded " + result + " bytes");
     }
 }
```



```java
//使用执行
new DownloadFilesTask().execute(url1, url2, url3);
```



#### 参数类型

异步任务有三个类型的参数：

+ Params：在执行时传入task中的参数类型
+ Progress：后台计算时publicsh的进度单位
+ Result：后台计算结果的类型

如果后台任务中不需要使用某个类型，可以将该类型设为Void，如

```java
private class MyTask extends AsyncTask<Void, Void, Void> { ... }
```



#### 4步

​	异步任务执行时，共经历了4个阶段：

1. onPreExecute()

   在task执行前运行于UI线程。通常用于设置task，如显示进度条。

2. doInBackground(Params...)

   在onPreExecute()结束后立即调用后台线程。该阶段用来执行会占用长时间的后台计算，此处返回的结果将传递到最后的阶段。在此阶段还可以使用`publishProgress(Progress...)`来发布一个或多个单位的进度，这些值将在onProgressUpdate(Progress...)阶段被展示在UI线程。

3. onProgressUpdate(Progress...)

   使用publishProgress(Progress...)后在UI线程调用。用来在后台计算过程中向用户展示进度。

4. onPostExecute(Result)

   后台计算结束后再UI线程调用。



#### 取消task

​	task可以在任意时间通过调用cancel(boolean)来取消，调用该方法将导致后续调用isCancelled()方法时返回true。在doInBackground(Object[])返回后，将回调onCancelled(Object)而不是onPostExecute(Object)。

> 为确保task可以尽快地取消，如果可能（如循环中），应在doInBackground(Objects[])中定期检查isCancelled()的返回值。



#### 线程规则

+ AsyncTask必须在UI线程中加载。在`JELLY_BEAN`中会自动完成。
+ task实例必须在UI线程中创建。
+ execute(Params...)必须在UI线程中调用。
+ 不要主动调用onPreExecute(),onPostExecute(Result),doInBackground(Prams...),onProgressUpdate(Progress...)。
+ task只能执行一次（试图执行第二次时会抛出异常）。



#### (*)执行顺序

+ AsyncTask一开始出现时，被连续执行在一个单一的后台线程中。

+ 从`DONUT`开始，更改为允许多task并行操作的线程池。

+ `HONEYCOMB`开始，task被执行在单一线程来避免由并行执行(parallel execution)导致的一般应用程序错误。

  如果需要使用并行执行，可结合[THREAD_POOL_EXECUTOR](https://developer.android.com/reference/android/os/AsyncTask.html#THREAD_POOL_EXECUTOR)来调用[executeOnExecutor(java.util.concurrent.Executor, Object[])](https://developer.android.com/reference/android/os/AsyncTask.html#executeOnExecutor(java.util.concurrent.Executor, Params...))。

> executeOnExecutor
>
> Executes the task with the specified parameters. The task returns itself (this) so that the caller can keep a reference to it.
>
> This method is typically used with THREAD_POOL_EXECUTOR to allow multiple tasks to run in parallel on a pool of threads managed by AsyncTask, however you can also use your own Executor for custom behavior.
>
> Warning: Allowing multiple tasks to run in parallel from a thread pool is generally not what one wants, because the order of their operation is not defined. For example, if these tasks are used to modify any state in common (such as writing a file due to a button click), there are no guarantees on the order of the modifications. Without careful work it is possible in rare cases for the newer version of the data to be over-written by an older one, leading to obscure data loss and stability issues. Such changes are best executed in serial; to guarantee such work is serialized regardless of platform version you can use this function with SERIAL_EXECUTOR.
>
> This method must be invoked on the UI thread.



## 为什么不在AsyncTask执行长时间操作

[Android AsyncTask for Long Running Operations](http://stackoverflow.com/questions/12797550/android-asynctask-for-long-running-operations)

​	主要问题点

+ AsyncTask缺乏与activity生命周期的联系

  AsyncTask不跟随Activity实例的生命周期。如在Activity中启动一个AsyncTask然后旋转设备，Activity将被销毁并创建一个新的实例。但AsyncTask不会被杀死，而是会存留到完成。一旦完成，task不会更新新Activity的UI，而是更新前一个已经不再显示的Activity实例，这将可能导致异常。

+ 内存泄漏

  将AsyncTask作为Activity的内部类来创建是一种很方便的使用方式：内部类可以直接访问外部类的任意字段。**然而，这意味着内部类将持有外部类实例的隐式引用。**在长时间的运行过程中，这将产生内存泄漏：如果AsyncTask持续了一段较长的时间，将保留Activity为alive，尽管Android因为它不再显示而想要处理它，即Activity无法被GC。

+ 从Android 3.2开始，AsyncTask默认使用的线程池只有一个线程：可能导致其他task无法运行。

  ​





