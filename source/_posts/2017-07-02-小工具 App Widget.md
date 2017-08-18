---
title: API Guides - App Widget 小部件
tags:
  - android
  - app widget
date: 2017-07-02 15:37:22
categories: 笔记
---

[App Widgets](https://developer.android.com/guide/topics/appwidgets/index.html#AppWidgetProvider)

[Widgets design](https://developer.android.com/design/patterns/widgets.html)

​	App 小部件是可以嵌入到其它应用(如主屏幕)并定期刷新的 view，而可以放置其它应用的 Widget 的应用组件叫做App Widget host。App Widget通过 App Widget provider 来发布。下面是一个音乐的 AppWidget：

![appwidget](https://ws3.sinaimg.cn/large/006tKfTcgy1finu4j2booj308o02sgln.jpg)

## 基本

​	创建一个 App Widget，需要：

+ AppWidgetProviderInfo 对象

  描述 App Widget 的元数据，如 layout、更新频率以及AppWidgetProvider类。这些数据都需要定义在 XML 中。

+ AppWidgetProvider 实现类

  AppWidgetProvider 定义了与 App Widget 对接的基于广播事件的基本方法，它的父类是 BroadcastReceiver。通过它，可以在 App Widget 刷新、enabled、disabled 或删除时收到广播。

+ View 布局

  在 XML 中为 App Widget 定义初始布局



​	此外，还可以实现一个 App Widget 配置 Activity。



## 在 Manifest 中声明一个App Widget

如：

```xml
<receiver android:name="ExampleAppWidgetProvider" >
    <intent-filter>
        <action android:name="android.appwidget.action.APPWIDGET_UPDATE" />
    </intent-filter>
    <meta-data android:name="android.appwidget.provider"
               android:resource="@xml/example_appwidget_info" />
</receiver>
```

​	其中，APPWIDGET_UPDATE广播必须显式声明的唯一一个广播，AppWidgetManager 将在必要时自动发送其他 App Widget 的广播到 AppWidgetProvider。\<meta-data> 元素用于指定 AppWidgetProviderInfo 资源：

+ android:name

  元数据名称，android.appwidget.provider 表明此数据为 AppWidgetProviderInfo 描述符

+ android:resource

  定位  AppWidgetProviderInfo 资源



## 添加 AppWidgetProviderInfo 元数据

​	AppWidgetProviderInfo 对象以 \<appwidget-provider> 元素定义在 XML 文件中。如：

```xml
<appwidget-provider xmlns:android="http://schemas.android.com/apk/res/android"
    android:minWidth="40dp"
    android:minHeight="40dp"
    android:updatePeriodMillis="86400000"
    android:previewImage="@drawable/preview"
    android:initialLayout="@layout/example_appwidget"
    android:configure="com.example.android.ExampleAppWidgetConfigure"
    android:resizeMode="horizontal|vertical"
    android:widgetCategory="home_screen">
</appwidget-provider>
```

​	主屏幕在放置 App Widget 时，是以一种网格的方式来确定大小的，为使小部件适用于多种设备，最小大小最好不要超过 4 X 4。

\<appwidget-provider> 支持以下属性

```xml
    <declare-styleable name="AppWidgetProviderInfo">
        <attr name="minWidth"/>
        <attr name="minHeight"/>
        <attr name="minResizeWidth" format="dimension"/>
        <attr name="minResizeHeight" format="dimension"/>
        <!-- 更新周期，0表示 AppWidget 将主动更新 -->
        <attr name="updatePeriodMillis" format="integer" />
        <attr name="initialLayout" format="reference" />
        <attr name="initialKeyguardLayout" format="reference" />
        <!-- AppWidget 包中的一个类名，用于启动配置界面 -->
        <attr name="configure" format="string" />
        <attr name="previewImage" format="reference" />
        <!-- The view id of the AppWidget subview which should be auto-advanced.
             by the widget's host. -->
        <attr name="autoAdvanceViewId" format="reference" />
        <!-- 支持的 resize 方式 -->
        <attr name="resizeMode" format="integer">
            <flag name="none" value="0x0" />
            <flag name="horizontal" value="0x1" />
            <flag name="vertical" value="0x2" />
        </attr>
        <!-- widget 显示位置 -->
        <attr name="widgetCategory" format="integer">
            <flag name="home_screen" value="0x1" />
            <flag name="keyguard" value="0x2" />
            <flag name="searchbox" value="0x4" />
        </attr>
    </declare-styleable>
```



## 创建 App Widget 布局

​	App Widget 布局基于 RemoteView，所以并不支持所有的 layout 和 view 控件。

​	RemoteView 支持的 layout 类：

- `FrameLayout`

- `LinearLayout`

- `RelativeLayout`

- `GridLayout`

  支持的 widget 类：


- `AnalogClock`

- `Button`

- `Chronometer`

- `ImageButton`

- `ImageView`

- `ProgressBar`

- `TextView`

- `ViewFlipper`

- `ListView`

- `GridView`

- `StackView`

- `AdapterViewFlipper`

  这些类的子类也不被支持。RemoteView 支持的还有  ViewStub。



## AppWidgetProvider 类的使用

​	 AppWidgetProvider 继承于 BroadcastReceiver，用于处理 App Widget 的广播。在相关的广播事件发生时，AppWidgetProvider 将触发以下回调：

+ onUpdate()

   根据 AppWidgetProviderInfo 的 updatePeriodMillis 属性定期更新App Widget。此方法在用户添加 App Widget 时也会回调。但是，如果声明了配置 Activity，则在用户添加 App Widget 时将不会调用此方法，因为将由配置 Activity 来负责在配置完成时进行第一次更新。

+ onAppWidgetOptionsChanged()

  App Widget 第一次放置或被 resize 时调用。可以在此回调中根据 widget 的大小范围来显示/隐藏内容。 getAppWidgetOptions() 方法可以获得大小范围，它返回的 Bundle 带有以下内容：

  + OPTION_APPWIDGET_MIN_WIDTH
  + OPTION_APPWIDGET_MIN_HEIGHT
  + OPTION_APPWIDGET_MAX_WIDTH
  + OPTION_APPWIDGET_MAX_HEIGHT

+ onDeleted(Context, int[])

+ onEnabled(Context)

  此方法只是 App Widget 添加第一个实例时回调（一个 App Widget 可以主屏幕中添加多个实例）。 

+ onDisabled(Context)

  从 App Widget host 中删除最后一个 App Widget 实例时调用。

+ onReceive(Context, Intent)

  在以上方法回调之前调用。一般不需要重写。



​	AppWidgetProvider 最重要的回调方法是 onUpdate()，如果需要接受用户交互事件，需要在此回调中注册事件处理。如，在 App Widget 的一个 button 被点击时启动一个 Activity

```java
public class ExampleAppWidgetProvider extends AppWidgetProvider {

    public void onUpdate(Context context, AppWidgetManager appWidgetManager, int[] appWidgetIds) {
        final int N = appWidgetIds.length;

        // Perform this loop procedure for each App Widget that belongs to this provider
      	// 为所有的 app widget 实例添加事件
        for (int i=0; i<N; i++) {
            int appWidgetId = appWidgetIds[i];

            // Create an Intent to launch ExampleActivity
            Intent intent = new Intent(context, ExampleActivity.class);
            PendingIntent pendingIntent = PendingIntent.getActivity(context, 0, intent, 0);

            // Get the layout for the App Widget and attach an on-click listener
            // to the button
            RemoteViews views = new RemoteViews(context.getPackageName(), R.layout.appwidget_provider_layout);
            views.setOnClickPendingIntent(R.id.button, pendingIntent);

            // Tell the AppWidgetManager to perform an update on the current app widget
          	// 个人猜测：每一次刷新都会使用新的 RemoteViews，说明刷新并不是直接对已放置的 view 进行
          	// 设置，而是进行了替换
            appWidgetManager.updateAppWidget(appWidgetId, views);
        }
    }
}
```



## 创建配置 Activity

​	 App Widget host 在创建 App Widget 时将自动启动配置 Activity 。

​	配置 Activity 同样需要在 Manifest 中声明，并添加 APPWIDGET_CONFIGURE 的 action，因为 App Widget host 将使用此 action 来启动 Activity。

```xml
<activity android:name=".ExampleAppWidgetConfigure">
    <intent-filter>
        <action android:name="android.appwidget.action.APPWIDGET_CONFIGURE"/>
    </intent-filter>
</activity>
```

​	然后，需要在 AppWidgetProviderInfo 的 XMl 文件中设置 configure 属性

```xml
<appwidget-provider xmlns:android="http://schemas.android.com/apk/res/android"
    ...
    android:configure="com.example.android.ExampleAppWidgetConfigure"
    ... >
</appwidget-provider>
```

​	在实现配置 Activity 时必须记住两件事情：

+ App Widget host 调用了配置 Activity ，配置 Activity 必须返回一个结果。该结果要包含启动 Activity 的Intent中的 App Widget 的 ID（ EXTRA_APPWIDGET_ID字段）。
+ onUpdate() 方法在 App Widget 创建时不会被调用，配置 Activity 需要负责第一次更新。



###  从配置 Activity 中更新 App Widget

大概步骤：

1. 获取 App Widget 的 ID

   ```Java
   Intent intent = getIntent();
   Bundle extras = intent.getExtras();
   if (extras != null) {
       mAppWidgetId = extras.getInt(
               AppWidgetManager.EXTRA_APPWIDGET_ID,
               AppWidgetManager.INVALID_APPWIDGET_ID);
   }
   ```

2. 执行 App Widget 配置

3. 配置完成时获取 AppWidgetManager 实例

   ```java
   AppWidgetManager appWidgetManager = AppWidgetManager.getInstance(context);
   ```

4. 更新 App Widget

   ```java
   RemoteViews views = new RemoteViews(context.getPackageName(),
   R.layout.example_appwidget);
   appWidgetManager.updateAppWidget(mAppWidgetId, views);
   ```

5. 创建返回 Intent，结束 Activity

   ```java
   Intent resultValue = new Intent();
   resultValue.putExtra(AppWidgetManager.EXTRA_APPWIDGET_ID, mAppWidgetId);
   setResult(RESULT_OK, resultValue);
   finish();
   ```

> 建议：在配置 Activity 第一次打开时，将 Activity result 设为 RESULT_CANCELED(在 onCreate 中调用setResult(RESULT_CANCELED))。通过这种方式，如果用户在完成之前退出 Activity，App Widget 将不会被添加。



## 在 App Widget 中使用集合

​	带 collection 的 App Widget 使用 RemoteViewsService 来显示基于远程数据(如content provider)的 collection。app widget 以以下 view 类型来展示 RemoteViewsServeice 提供的数据：

- ListView

- GridView

- StackView

  堆叠卡片视图（与名片盒相似），用户可以滑动来切换前一个/后一个卡片

- AdapterViewFlipper

  ​App Widget 需要使用一个 Adapter 来将数据绑定到独立的视图中，这个 Adapter 为 RemoteViewsFactory，它是一般 Adapter 的一个简易版封装。当请求 collection 中的一个指定 item 时，RemoteViewsFactory 创建并以 RemoteViews 对象的形式返回 collection 中的item。**为了使 app widget 支持 collection 视图，必须实现 RemoteViewsService 和 RemoteViewsFactory。**

  ​RemoteViewsService 是一个 service，它允许 remote adapter 请求 RemoteViews 对象。RemoteViewsFactory 是一个介于 collection 视图和视图底层数据之间的 adapter 的接口。

  ​如 [StackView Widget sample](https://android.googlesource.com/platform/development/+/master/samples/StackWidget/)中的实现：

```java
public class StackWidgetService extends RemoteViewsService {
    @Override
    public RemoteViewsFactory onGetViewFactory(Intent intent) {
        return new StackRemoteViewsFactory(this.getApplicationContext(), intent);
    }
}

class StackRemoteViewsFactory implements RemoteViewsService.RemoteViewsFactory {

//... include adapter-like methods here. See the StackView Widget sample.

}
```

​	 StackView Widget sample 界面：

![StackWidget](https://ws2.sinaimg.cn/large/006tKfTcgy1finu4i4wbfj305c05cwec.jpg)

### 实现带 collection 的appwidget

​	以下内容是除一般 app widget 步骤外的工作

#### Manifest

​	声明 service，如：

```xml
<service android:name="MyWidgetService"
...
android:permission="android.permission.BIND_REMOTEVIEWS" />
```

​	"MyWidgetService"为 RemoteViewsService 的子类，"android.permission.BIND_REMOTEVIEWS"防止其他应用访问 app widget 的数据。



#### 布局

​	需要在 layout 文件中包含一个 collection 视图。如：

```xml
<?xml version="1.0" encoding="utf-8"?>

<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <StackView xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@+id/stack_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:gravity="center"
        android:loopViews="true" />
    <TextView xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@+id/empty_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:gravity="center"
        android:background="@drawable/widget_item_background"
        android:textColor="#ffffff"
        android:textStyle="bold"
        android:text="@string/empty_view_text"
        android:textSize="20sp" />
</FrameLayout>
```

​	除了用于整个 app widget 的 layout 文件，还必须为 collection 的 item 创建另外的 layout 文件。



#### AppWidgetProvider

​	 AppWidgetProvider 主要的不同在于必须在 onUpdate() 中调用 setRemoteAdapter()。setRemoteAdapter() 用于告诉 collection 视图从哪里获取数据，RemoteViewsService 将返回 RemoteViewsFactory 的实现来为 widget 提供适当的数据。调用此方法的时候，必须提供一个指向 RemoteViewsService 实现的 intent 和 app widget 的ID。如：

```java
public void onUpdate(Context context, AppWidgetManager appWidgetManager,
int[] appWidgetIds) {
    // update each of the app widgets with the remote adapter
    for (int i = 0; i < appWidgetIds.length; ++i) {

        // Set up the intent that starts the StackViewService, which will
        // provide the views for this collection.
        Intent intent = new Intent(context, StackWidgetService.class);
        // Add the app widget ID to the intent extras.
        intent.putExtra(AppWidgetManager.EXTRA_APPWIDGET_ID, appWidgetIds[i]);
        intent.setData(Uri.parse(intent.toUri(Intent.URI_INTENT_SCHEME)));
        // Instantiate the RemoteViews object for the app widget layout.
        RemoteViews rv = new RemoteViews(context.getPackageName(), R.layout.widget_layout);
        // Set up the RemoteViews object to use a RemoteViews adapter.
        // This adapter connects
        // to a RemoteViewsService  through the specified intent.
        // This is how you populate the data.
        rv.setRemoteAdapter(appWidgetIds[i], R.id.stack_view, intent);

        // The empty view is displayed when the collection has no items.
        // It should be in the same layout used to instantiate the RemoteViews
        // object above.
        rv.setEmptyView(R.id.stack_view, R.id.empty_view);

        //
        // Do additional processing specific to this app widget...
        //

        appWidgetManager.updateAppWidget(appWidgetIds[i], rv);
    }
    super.onUpdate(context, appWidgetManager, appWidgetIds);
}
```



#### RemoteViewsService

​	RemoteViewsService 子类提供填充远程 collection 视图的 RemoteViewsFactory。具体的使用步骤如下：

1. 子类化 RemoteViewsService。RemoteViewsService 是远程 adapter 请求 RemoteViews 的服务。
2. 在 RemoteViewsService 的子类中，需要包含一个实现了 RemoteViewsFactory 接口的类。

#### RemoteViewsFactory 接口

​	RemoteViewsFactory 接口实现中最重要的两个方法是 onCreate() 和 getViewAt()。

​	系统在第一次创建 factory 时调用 onCreate()，应该在此处配置数据源的连接或 cursor。如：

```java
class StackRemoteViewsFactory implements
RemoteViewsService.RemoteViewsFactory {
    private static final int mCount = 10;
    private List<WidgetItem> mWidgetItems = new ArrayList<WidgetItem>();
    private Context mContext;
    private int mAppWidgetId;

    public StackRemoteViewsFactory(Context context, Intent intent) {
        mContext = context;
        mAppWidgetId = intent.getIntExtra(AppWidgetManager.EXTRA_APPWIDGET_ID,
                AppWidgetManager.INVALID_APPWIDGET_ID);
    }

    public void onCreate() {
        // In onCreate() you setup any connections / cursors to your data source. Heavy lifting,
        // for example downloading or creating content etc, should be deferred to onDataSetChanged()
        // or getViewAt(). Taking more than 20 seconds in this call will result in an ANR.
        for (int i = 0; i < mCount; i++) {
            mWidgetItems.add(new WidgetItem(i + "!"));
        }
        ...
    }
...
```

​	而 getViewAt() 方法返回一个对应于指定位置数据的 RemoteViews 对象。如：

```java
public RemoteViews getViewAt(int position) {

    // Construct a remote views item based on the app widget item XML file,
    // and set the text based on the position.
    RemoteViews rv = new RemoteViews(mContext.getPackageName(), R.layout.widget_item);
    rv.setTextViewText(R.id.widget_item, mWidgetItems.get(position).text);

    ...
    // Return the remote views object.
    return rv;
}
```



#### 为单独的 item 添加行为

​	在上面使用 AppWidgetProvider 的一般使用中，可以通过 setOnClickPendingIntent() 为一个对象设置点击行为，但这种方式无法用于独立的 collection item 的子视图中。

​	以 [StackView Widget sample](https://android.googlesource.com/platform/development/+/master/samples/StackWidget/)  说明（点击 top view 时，弹出提示）：

- StackWidgetProvider ( AppWidgetProvider 子类) 创建一个自定义 action 为 TOAST_ACTION 的 pending intent

- 用户触摸 view 时，触发 intent 发出广播 TOAST_ACTION

- 在 StackWidgetProvider 的 onReceive() 方法中拦截该广播

  > ​在带 collection 的 widget 中，如果为每一个独立的 item 设置 pendingIntent，将耗费很多的资源，所以一般是为 collection 视图设置一个 PendingIntent，然后在 RemoteViewsFactory 的 getViewAt(...) 方法中调用 setOnClickFillInIntent(...) 为每一个 item 设置一个 fillInIntent 以区分每一个 item。 

#### 配置 pending intent 模板

​	单独的 collection item 无法配置自己的pending intent。采用的方案是，由 collection 作为一个整体来设置一个 pending intent 模板，而单独的 item 设置 fill-in intent 来创建自己唯一的行为。

```java
public class StackWidgetProvider extends AppWidgetProvider {
    public static final String TOAST_ACTION = "com.example.android.stackwidget.TOAST_ACTION";
    public static final String EXTRA_ITEM = "com.example.android.stackwidget.EXTRA_ITEM";

    ...

    // Called when the BroadcastReceiver receives an Intent broadcast.
    // Checks to see whether the intent's action is TOAST_ACTION. If it is, the app widget
    // displays a Toast message for the current item.
    @Override
    public void onReceive(Context context, Intent intent) {
        AppWidgetManager mgr = AppWidgetManager.getInstance(context);
        if (intent.getAction().equals(TOAST_ACTION)) {
            int appWidgetId = intent.getIntExtra(AppWidgetManager.EXTRA_APPWIDGET_ID,
                AppWidgetManager.INVALID_APPWIDGET_ID);
            int viewIndex = intent.getIntExtra(EXTRA_ITEM, 0);
            Toast.makeText(context, "Touched view " + viewIndex, Toast.LENGTH_SHORT).show();
        }
        super.onReceive(context, intent);
    }

    @Override
    public void onUpdate(Context context, AppWidgetManager appWidgetManager, int[] appWidgetIds) {
        // update each of the app widgets with the remote adapter
        for (int i = 0; i < appWidgetIds.length; ++i) {

            // Sets up the intent that points to the StackViewService that will
            // provide the views for this collection.
            Intent intent = new Intent(context, StackWidgetService.class);
            intent.putExtra(AppWidgetManager.EXTRA_APPWIDGET_ID, appWidgetIds[i]);
            // When intents are compared, the extras are ignored, so we need to embed the extras
            // into the data so that the extras will not be ignored.
            intent.setData(Uri.parse(intent.toUri(Intent.URI_INTENT_SCHEME)));
            RemoteViews rv = new RemoteViews(context.getPackageName(), R.layout.widget_layout);
            rv.setRemoteAdapter(appWidgetIds[i], R.id.stack_view, intent);

            // The empty view is displayed when the collection has no items. It should be a sibling
            // of the collection view.
            rv.setEmptyView(R.id.stack_view, R.id.empty_view);

            // This section makes it possible for items to have individualized behavior.
            // It does this by setting up a pending intent template. Individuals items of a collection
            // cannot set up their own pending intents. Instead, the collection as a whole sets
            // up a pending intent template, and the individual items set a fillInIntent
            // to create unique behavior on an item-by-item basis.
            Intent toastIntent = new Intent(context, StackWidgetProvider.class);
            // Set the action for the intent.
            // When the user touches a particular view, it will have the effect of
            // broadcasting TOAST_ACTION.
            toastIntent.setAction(StackWidgetProvider.TOAST_ACTION);
            toastIntent.putExtra(AppWidgetManager.EXTRA_APPWIDGET_ID, appWidgetIds[i]);
            intent.setData(Uri.parse(intent.toUri(Intent.URI_INTENT_SCHEME)));
            PendingIntent toastPendingIntent = PendingIntent.getBroadcast(context, 0, toastIntent,
                PendingIntent.FLAG_UPDATE_CURRENT);
            rv.setPendingIntentTemplate(R.id.stack_view, toastPendingIntent);

            appWidgetManager.updateAppWidget(appWidgetIds[i], rv);
        }
    super.onUpdate(context, appWidgetManager, appWidgetIds);
    }
}
```



#### 设置 fill-in intent

​	RemoteViewsFactory 必须为 collection 中的每一个 item 设置一个 fill-in intent。fill-in intent 结合 PendingIntent 模板为确定点击 item 时的最终意图。

​	如：

```java
public class StackWidgetService extends RemoteViewsService {
    @Override
    public RemoteViewsFactory onGetViewFactory(Intent intent) {
        return new StackRemoteViewsFactory(this.getApplicationContext(), intent);
    }
}

class StackRemoteViewsFactory implements RemoteViewsService.RemoteViewsFactory {
    private static final int mCount = 10;
    private List<WidgetItem> mWidgetItems = new ArrayList<WidgetItem>();
    private Context mContext;
    private int mAppWidgetId;

    public StackRemoteViewsFactory(Context context, Intent intent) {
        mContext = context;
        mAppWidgetId = intent.getIntExtra(AppWidgetManager.EXTRA_APPWIDGET_ID,
                AppWidgetManager.INVALID_APPWIDGET_ID);
    }

    // Initialize the data set.
        public void onCreate() {
            // In onCreate() you set up any connections / cursors to your data source. Heavy lifting,
            // for example downloading or creating content etc, should be deferred to onDataSetChanged()
            // or getViewAt(). Taking more than 20 seconds in this call will result in an ANR.
            for (int i = 0; i < mCount; i++) {
                mWidgetItems.add(new WidgetItem(i + "!"));
            }
           ...
        }
        ...

        // Given the position (index) of a WidgetItem in the array, use the item's text value in
        // combination with the app widget item XML file to construct a RemoteViews object.
        public RemoteViews getViewAt(int position) {
            // position will always range from 0 to getCount() - 1.

            // Construct a RemoteViews item based on the app widget item XML file, and set the
            // text based on the position.
            RemoteViews rv = new RemoteViews(mContext.getPackageName(), R.layout.widget_item);
            rv.setTextViewText(R.id.widget_item, mWidgetItems.get(position).text);

            // Next, set a fill-intent, which will be used to fill in the pending intent template
            // that is set on the collection view in StackWidgetProvider.
            Bundle extras = new Bundle();
            extras.putInt(StackWidgetProvider.EXTRA_ITEM, position);
            Intent fillInIntent = new Intent();
            fillInIntent.putExtras(extras);
            // Make it possible to distinguish the individual on-click
            // action of a given item
            rv.setOnClickFillInIntent(R.id.widget_item, fillInIntent);

            ...

            // Return the RemoteViews object.
            return rv;
        }
    ...
    }
```



### 使 collection 数据保持刷新

​	下面的图说明了发生更新时的流程

![appwidget_collections](https://ws1.sinaimg.cn/large/006tKfTcgy1finu4iywmej30f70h4gmn.jpg)









