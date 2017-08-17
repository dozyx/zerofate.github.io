---
title: API Guides - MediaPlayer
tags:
  - android
  - media
date: 2016-10-09 15:37:22
categories: 笔记
---

[API Guide - MediaPlayer](https://developer.android.com/guide/topics/media/mediaplayer.html) 

​	Android的多媒体框架支持播放多种常见类型的音频、视频、图片媒体文件。存储在应用内部、独立文件系统或数据流中的媒体文件，都是使用MediaPlayer APIs进行播放。



### 基础

[MediaPlayer ](https://developer.android.com/reference/android/media/MediaPlayer.html)  播放声音和视频的基本API

[AudioManager](https://developer.android.com/reference/android/media/AudioManager.html)  管理设备上的音频源和音频输出



### Manifest 声明

网络权限：

`<uses-permissionandroid:name="android.permission.INTERNET" />`

休眠权限：

`<uses-permissionandroid:name="android.permission.WAKE_LOCK" />`



### 使用MediaPlayer

MediaPlayer对象可以以尽少的步骤来捕获、解码、播放音视频。它支持不同的媒体源，包括：

+ 本地资源
+ Internal URIs
+ External URLs（流）

支持格式：

[AndroidSupported Media Formats](https://developer.android.com/guide/appendix/media-formats.html) 



#### 示例

播放 raw 资源

```java
MediaPlayer mediaPlayer = MediaPlayer.create(context, R.raw.sound_file_1);
mediaPlayer.start(); // no need to call prepare(); create() does that for you
```

播放本地URI

```java
Uri myUri = ....; // initialize Uri here
MediaPlayer mediaPlayer = new MediaPlayer();
mediaPlayer.setAudioStreamType(AudioManager.STREAM_MUSIC);
mediaPlayer.setDataSource(getApplicationContext(), myUri);
mediaPlayer.prepare();
mediaPlayer.start();
```

通过HTTP流播放远程的URL

```java
String url = "http://........"; // your URL here
MediaPlayer mediaPlayer = new MediaPlayer();
mediaPlayer.setAudioStreamType(AudioManager.STREAM_MUSIC);
mediaPlayer.setDataSource(url);
mediaPlayer.prepare(); // might take long! (for buffering, etc)
mediaPlayer.start();
```



#### 注意点

+ Asynchronous Preparation

  调用prepare()方法将执行较长的时间，因为它可能需要捕获和解码媒体数据，因此永远不要在UI线程中进行调用。所以应当启动另一个线程对MediaPlayer进行prepare并在结束时通知主线程。虽然可以自行编写线程逻辑，但更为通用的做法是使用prepareAsync()方法来完成这个任务，当media完成准备后，MediaPlayer.OnPreparedListener中的onPrepared()方法会立即被调用。

+ Managing State

  MediaPlayer是基于状态的，在编写代码过程中需要时刻注意MediaPlayer的内部状态，因为一些操作只有在指定的状态下才能执行，如果在错误的状态执行了一些操作，系统将抛出一些异常或其他不理想的状态。

+ Releasing the MediaPlayer

  MediaPlayer会消耗宝贵的系统资源

  ```java
  mediaPlayer.release();
  mediaPlayer = null;
  ```



### 在服务中使用MediaPlayer

​	如果需要在后台播放媒体，则需要启动一个Service并在Service里控制MediaPalyer实例。

​	注意事项及建议：

+ Running asynchronously

  跟Activity一样，Service中的所有工作默认都是在一个单线程中完成，事实上，如果Activity和Service运行在同一个应用上，则默认使用一样的线程（主线程）。
  举例：在主线程中使用MediaPlayer时，应当调用prepareAsync()而不是prepare()，并实现MediaPlayer.OnPreparedListener来通知准备完成并可以开始播放。

  ```java
  public class MyService extends Service implements MediaPlayer.OnPreparedListener {
      private static final String ACTION_PLAY = "com.example.action.PLAY";
      MediaPlayer mMediaPlayer = null;

      public int onStartCommand(Intent intent, int flags, int startId) {
          ...
          if (intent.getAction().equals(ACTION_PLAY)) {
              mMediaPlayer = ... // initialize it here
              mMediaPlayer.setOnPreparedListener(this);
              mMediaPlayer.prepareAsync(); // prepare async to not block main thread
          }
      }

      /** Called when MediaPlayer is ready */
      public void onPrepared(MediaPlayer player) {
          player.start();
      }

  ```

+ Handling asynchronous errors

  当错误产生时，MediaPlayer进入error状态，如果需要复用error状态的MediaPlayer对象，可以调用reset()来讲对象恢复到idle状态。

  ```java
  public class MyService extends Service implements MediaPlayer.OnErrorListener {
      MediaPlayer mMediaPlayer;

      public void initMediaPlayer() {
          // ...initialize the MediaPlayer here...

          mMediaPlayer.setOnErrorListener(this);
      }

      @Override
      public boolean onError(MediaPlayer mp, int what, int extra) {
          // ... react appropriately ...
          // The MediaPlayer has moved to the Error state, must be reset!
      }
  }
  ```

+ Using wake locks

  应当在真正必要时才使用wake locks。

  为确保CPU在MediaPlayer播放时保持运行，在初始化MediaPlayer时调用setWakeMode()方法，这样，MediaPlayer在播放时将维持specified lock并在paused或stoped时释放lock

  ```java
  mMediaPlayer = new MediaPlayer();
  // ... other initialization here ...
  mMediaPlayer.setWakeMode(getApplicationContext(), PowerManager.PARTIAL_WAKE_LOCK);
  ```

  得到的wake lock只保证CPU维持唤醒，如果使用wifi则需要同时持有wifilock。

  ```java
  WifiLock wifiLock = ((WifiManager) getSystemService(Context.WIFI_SERVICE))
      .createWifiLock(WifiManager.WIFI_MODE_FULL, "mylock");

  wifiLock.acquire();
  ```

  paused或stoped或者不再使用时释放

  ```java
  wifiLock.release();
  ```

+ Performing cleanup

  ```java
  public class MyService extends Service {
     MediaPlayer mMediaPlayer;
     // ...

     @Override
     public void onDestroy() {
         if (mMediaPlayer != null) mMediaPlayer.release();
     }
  }
  ```



### 从 Content Resolver 获取媒体文件

```java
ContentResolver contentResolver = getContentResolver();
Uri uri = android.provider.MediaStore.Audio.Media.EXTERNAL_CONTENT_URI;
Cursor cursor = contentResolver.query(uri, null, null, null, null);
if (cursor == null) {
    // query failed, handle error.
} else if (!cursor.moveToFirst()) {
    // no media on the device
} else {
    int titleColumn = cursor.getColumnIndex(android.provider.MediaStore.Audio.Media.TITLE);
    int idColumn = cursor.getColumnIndex(android.provider.MediaStore.Audio.Media._ID);
    do {
       long thisId = cursor.getLong(idColumn);
       String thisTitle = cursor.getString(titleColumn);
       // ...process entry...
    } while (cursor.moveToNext());
}
```

播放

```java
long id = /* retrieve it from somewhere */;
Uri contentUri = ContentUris.withAppendedId(
        android.provider.MediaStore.Audio.Media.EXTERNAL_CONTENT_URI, id);

mMediaPlayer = new MediaPlayer();
mMediaPlayer.setAudioStreamType(AudioManager.STREAM_MUSIC);
mMediaPlayer.setDataSource(getApplicationContext(), contentUri);

// ...prepare and start...
```









