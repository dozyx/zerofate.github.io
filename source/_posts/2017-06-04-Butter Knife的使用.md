---
title: Butter Knife - 官方指导学习
tags:
  - android
  - Butter Knife
date: 2017-06-04 15:37:22
categories: 笔记
---

[Butter Knife](http://jakewharton.github.io/butterknife/)

> 以下内容主要翻译自 Butter Knife 的官方发表页，主要用于自己理解记忆 Butter Knife 用法。
>
> **Remember: A butter knife is like a dagger only infinitely less sharp.**

Butter Knife，黄油刀，它的用途在于

+ 对 filed 使用 @BindView 来消除 findViewById
+ 对 list 或 array 中的多个 view 进行分组，然后对它们统一进行actions、setters或properties 操作。
+ 使用 @OnClick 或其他注解方法来消除匿名内部类的监听器
+ 对 filed 使用资源注解来消除资源查找

工程引用(gradle)

```groovy
dependencies {
  compile 'com.jakewharton:butterknife:8.6.0'
  annotationProcessor 'com.jakewharton:butterknife-compiler:8.6.0'
}
```

### 最常见用法

最常用的用法是通过使用 @BindView 注解和一个 view id 来自动获取布局中对应的 view。

示例代码：

```java
class ExampleActivity extends Activity {
  @BindView(R.id.title) TextView title;
  @BindView(R.id.subtitle) TextView subtitle;
  @BindView(R.id.footer) TextView footer;

  @Override public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.simple_activity);
    ButterKnife.bind(this);
    // TODO Use fields...
  }
}
```

需要注意的地方有两点：一个是注解声明，一个是ButterKnife.bind()。通过这种方式可以减少大量的 findViewById() 代码。上面的代码等同于

```java
public void bind(ExampleActivity activity) {
  activity.subtitle = (android.widget.TextView) activity.findViewById(2130968578);
  activity.footer = (android.widget.TextView) activity.findViewById(2130968579);
  activity.title = (android.widget.TextView) activity.findViewById(2130968577);
}
```

除了上面的用法外，Butter Knife 还有一些其他的用法。



### 绑定资源

Butter Knife 支持使用 @BindBool、@BindColor、@BindDimen、@BindDrawable、@BindInt、@BindString 来绑定预置的资源。

示例代码：

```java
class ExampleActivity extends Activity {
  @BindString(R.string.title) String title;
  @BindDrawable(R.drawable.graphic) Drawable graphic;
  @BindColor(R.color.red) int red; // int or ColorStateList field
  @BindDimen(R.dimen.spacer) Float spacer; // int (for pixel size) or float (for exact value) field
  // ...
}
```



### 非 Activity 绑定

通过提供根 view，我们对任意对象进行绑定。

#### Fragment

示例代码：

```java
public class FancyFragment extends Fragment {
  @BindView(R.id.button1) Button button1;
  @BindView(R.id.button2) Button button2;

  @Override public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
    View view = inflater.inflate(R.layout.fancy_fragment, container, false);
    ButterKnife.bind(this, view);
    // TODO Use fields...
    return view;
  }
}
```

>  **从这里可以看出，@BindView 的用法主要分两步：声明 + 静态方法绑定。**

另一种用法是简化 list adapter 里的 view holder 模式。

#### Adapter

示例代码：

```java
public class MyAdapter extends BaseAdapter {
  @Override public View getView(int position, View view, ViewGroup parent) {
    ViewHolder holder;
    if (view != null) {
      holder = (ViewHolder) view.getTag();
    } else {
      view = inflater.inflate(R.layout.whatever, parent, false);
      holder = new ViewHolder(view);
      view.setTag(holder);
    }

    holder.name.setText("John Doe");
    // etc...

    return view;
  }

  static class ViewHolder {
    @BindView(R.id.title) TextView name;
    @BindView(R.id.job_title) TextView jobTitle;

    public ViewHolder(View view) {
      ButterKnife.bind(this, view);
    }
  }
}
```



#### 其他绑定 API

+ 绑定任意将 Activity 作为根 view 的对象。如 MVC 模式中可以使用 ButterKnife.bind(this, activity) 来绑定 controller。
+ 使用 ButterKnife.bind(this) 来绑定 view 的 children。如果在布局中使用了 \<merge> 标签，可以在 view 的构造函数中进行 inflate 后立即调用此方法。或者，在使用 XML 进行 inflate 的自定义 view 时，在 onFInishInflate 回调中调用。

这部分翻译得不太好，下面是原文：

> Other provided binding APIs:
>
> - Bind arbitrary objects using an activity as the view root. If you use a pattern like MVC you can bind the controller using its activity with `ButterKnife.bind(this, activity)`.
> - Bind a view's children into fields using `ButterKnife.bind(this)`. If you use `<merge>` tags in a layout and inflate in a custom view constructor you can call this immediately after. Alternatively, custom view types inflated from XML can use it in the `onFinishInflate()` callback.



### view 列表

将多个 view 分组为一个 List 或 array。

```java
@BindViews({ R.id.first_name, R.id.middle_name, R.id.last_name })
List<EditText> nameViews;
```

使用 apply 方法一次性对 list 中的所有 view 采取操作

```java
ButterKnife.apply(nameViews, DISABLE);
ButterKnife.apply(nameViews, ENABLED, false);
```

实现 Action 接口

```java
static final ButterKnife.Action<View> DISABLE = new ButterKnife.Action<View>() {
  @Override public void apply(View view, int index) {
    view.setEnabled(false);
  }
};
static final ButterKnife.Setter<View, Boolean> ENABLED = new ButterKnife.Setter<View, Boolean>() {
  @Override public void set(View view, Boolean value, int index) {
    view.setEnabled(value);
  }
};
```

Android 的 Property 同样可以直接与 apply 方法一起使用

```java
ButterKnife.apply(nameViews, View.ALPHA, 0.0f);
```



### 监听器绑定

示例代码：

```java
@OnClick(R.id.submit)
public void submit(View view) {
  // TODO submit data to server...
}
```

监听器中的参数是可选的，也可以写成

```java
@OnClick(R.id.submit)
public void submit() {
  // TODO submit data to server...
}
```

还可以指定类型，ButterKnife 会进行类型强制转换

```java
@OnClick(R.id.submit)
public void sayHi(Button button) {
  button.setText("Hello!");
}
```

对多个 id 使用单个绑定

```java
@OnClick({ R.id.door1, R.id.door2, R.id.door3 })
public void pickDoor(DoorView door) {
  if (door.hasPrizeBehind()) {
    Toast.makeText(this, "You win!", LENGTH_SHORT).show();
  } else {
    Toast.makeText(this, "Try again", LENGTH_SHORT).show();
  }
}
```

自定义的 view 可以不指定 id 来进行监听器绑定

```java
public class FancyButton extends Button {
  @OnClick
  public void onClick() {
    // TODO do something!
  }
}
```



### 绑定重置

Fragment  中 view 的生命周期与 Activity 不同。在 fragment 的 onCreateView 中绑定的同时，还需要在 onDestoryView中将 view 设为 null。Butter Knife 在绑定时会返回一个 Unbinder 实例，我们需要在合适的生命周期回调中调用它的 unbind 方法。

示例代码：

```java
public class FancyFragment extends Fragment {
  @BindView(R.id.button1) Button button1;
  @BindView(R.id.button2) Button button2;
  private Unbinder unbinder;

  @Override public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
    View view = inflater.inflate(R.layout.fancy_fragment, container, false);
    unbinder = ButterKnife.bind(this, view);
    // TODO Use fields...
    return view;
  }

  @Override public void onDestroyView() {
    super.onDestroyView();
    unbinder.unbind();
  }
}
```



### 非强制绑定

默认的，@Bind 和监听器绑定都要求目标 view 必须存在，否则将抛出异常。不过，可以通过使用 @Nuallable注解修饰字段和使用 @Optional 注解修饰方法来避免这个问题。

示例代码：

```java
@Nullable @BindView(R.id.might_not_be_there) TextView mightNotBeThere;

@Optional @OnClick(R.id.maybe_missing) void onMaybeMissingClicked() {
  // TODO ...
}
```



### Multi-metod 监听器

示例代码：

```java
@OnItemSelected(R.id.list_view)
void onItemSelected(int position) {
  // TODO ...
}

@OnItemSelected(value = R.id.maybe_missing, callback = NOTHING_SELECTED)
void onNothingSelected() {
  // TODO ...
}
```

`void onItemSelected(AdapterView<?> parent, View view, int position, long id)`里的参数均可以使用。

**value 用于指定 view 的id，callback 用于指定处理的回调方法，可以设置的 callback 被定义为了 enum 类型**

### 其他

Butter Knife 同样包含了 findById 用法。

示例代码：

```java
View view = LayoutInflater.from(context).inflate(R.layout.thing, null);
TextView firstName = ButterKnife.findById(view, R.id.first_name);
TextView lastName = ButterKnife.findById(view, R.id.last_name);
ImageView photo = ButterKnife.findById(view, R.id.photo);
```