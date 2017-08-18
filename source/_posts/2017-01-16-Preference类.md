---
title: Prefeence 类
tags:
  - android
  - preference
date: 2017-01-16 15:37:22
categories: 笔记
---

java.lang.Object
   ↳	android.preference.Preference
Known Direct Subclasses
​	DialogPreference, PreferenceGroup, RingtonePreference, TwoStatePreference
Known Indirect Subclasses
​	CheckBoxPreference, EditTextPreference, ListPreference, MultiSelectListPreference, PreferenceCategory, 				   PreferenceScreen, SwitchPreference

​	Preference 表示的是一个基本的 Preference UI 构件块，PreferenceActivity 或 PreferenceFragment 将以 ListView 的形式显示出来。该类提供了显示的 View 并与一个用来存储偏好数据的 SharedPreference 关联。通过自定义preferenceStyle属性，可以将 Preference 设置为使用自定义的 style，默认的主题只指定了layout为preference.xml。

​	该类包含了一个用于SharedPreference 的Key，其子类决定如何存储它的值。

## Preference 布局

**preference.xml**

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android" 
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:minHeight="?android:attr/listPreferredItemHeight"
    android:gravity="center_vertical"
    android:paddingEnd="?android:attr/scrollbarSize"
    android:background="?android:attr/selectableItemBackground" >

    <ImageView
        android:id="@+android:id/icon"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        />

    <RelativeLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginStart="15dip"
        android:layout_marginEnd="6dip"
        android:layout_marginTop="6dip"
        android:layout_marginBottom="6dip"
        android:layout_weight="1">

        <TextView android:id="@+android:id/title"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:singleLine="true"
            android:textAppearance="?android:attr/textAppearanceLarge"
            android:ellipsize="marquee"
            android:fadingEdge="horizontal" />

        <TextView android:id="@+android:id/summary"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_below="@android:id/title"
            android:layout_alignStart="@android:id/title"
            android:textAppearance="?android:attr/textAppearanceSmall"
            android:textColor="?android:attr/textColorSecondary"
            android:maxLines="4" />

    </RelativeLayout>

    <!-- Preference should place its actual preference widget here. -->
    <LinearLayout android:id="@+android:id/widget_frame"
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        android:gravity="center_vertical"
        android:orientation="vertical" />

</LinearLayout>

```

显示效果如下：

![preference](https://ws1.sinaimg.cn/large/006tKfTcgy1finv0g6tdqj307i01mt8l.jpg)

最左侧为一个 icon，右侧为一个widget_frame可自定修改，通过 Preference 的 setWidgetLayoutResource 方法可以修改其布局，通常 Preference的子类就是通过修改该处布局来实现自定义。如果要修改整个 Preference 的布局，也可以使用 setLayoutResource 来设置为自定义的布局，但布局文件中通常要包含 widget_frame 的 id，同样的，还要有title和summary。

## Preference 属性

​	Preference 可以定义在 XML 文件中，它支持的属性有：

+ android:icon
+ android:key
+ android:title
+ android:summary
+ android:order  ： 如果没指定顺序，将按字母排列。**通过设置order可以在已有的Preference中间进行插入，注意需要调用 PreferenceGroup 的 setOrderingAsAdded(false)。**
+ android:fragment ： 如在 PreferenceActivity 中使用，将在点击 item 时启动一个新的 PreferenceFragment
+ android:layout：修改Preference 布局
+ android:widgetLayout：修改Preference 的widget_frame 布局
+ android:enabled：设置Preference 使能
+ android:selectable：Preference 是否可选择
+ android:dependency：**该Preference 所依赖的另一Preference的Key**，如果依赖的Preference未设置或处于关闭状态，则该Preference不可用
+ android:persistent：**是否将值存放到 SharedPreference中**
+ android:defaultValue：类型可以为string|boolean|integer|reference|float
+ android:shouldDisableView：当 Preference 为disabled 状态时，是否其 view 也为disabled





## 默认值

​	Preference的默认值可以在 android:defaultValue 中定义，其子类重写以下方法来实现获取

```java
	protected Object onGetDefaultValue(TypedArray a, int index) {
        return null;
    }
```

​	a 是属性的集合，可以从中读取出对应属性的值，index 表示的是 defaultValue 属性在 a 中的索引值。该方法主要作用在于根据默认值类型的不同分别从 a 中读取，这就要求子类来确保类型的正确性。

​	如CheckBoxPreference的父类 TwoStatePreference 

```java
	@Override
    protected Object onGetDefaultValue(TypedArray a, int index) {
        return a.getBoolean(index, false);
    }
```

## 数据持久化

​	Preference 的 isPersistent() 方法用于判断该 Preference 是否支持数据持久化，该方法只是简单的返回了 mPresistent  变量，mPresistent 默认为true，即Preference默认是支持数据持久化的。不过在 Preference 内部判断时还需要结合 key 是否存在来进行判断。

​	Preference 中提供了一系列的 persistXXX() 方法用于持久化保存数据和getPersistXXX() 方法用于读取，如  persistBoolean(boolean value)、persistFloat(float value)等。以下为persistInt 的源码部分

```java
    protected boolean persistInt(int value) {
        if (shouldPersist()) {
            if (value == getPersistedInt(~value)) {//~value为默认值
                // It's already there, so the same as persisting
                return true;
            }        
            SharedPreferences.Editor editor = mPreferenceManager.getEditor();
            editor.putInt(mKey, value);
            tryCommit(editor);
            return true;
        }
        return false;
    }
```

### onSetInitialValue

​	Preference 的子类重写 onSetInitialValue 来设置初始状态

### 主动修改偏好值

​	PreferenceManager 提供了一个静态方法 getDefaultSharedPreferences(Context) 可以直接获取preference 框架使用的指向默认文件的SharedPreference。

## BaseSavedState

​	Preference 中还有 onSaveInstanceState 和 onRestoreInstanceState 方法，这两个方法在 Fragment 和 Activity对应方法执行时回调。所以这部分需要结合 Fragment 和 Activity 相关部分研究。（待整理） 

## 自定义Preference

​	如果只是简单调整布局，只需要调用 setLayoutResource 和 setWidgetLayoutResource 两个方法修改布局即可，需要注意的是控件的 id 需要使用 andorid 原生的 id，如title、widget_frame；如果需要修改为自己的 widget_frame 并修改其行为，可以实现 Preference 的子类。

​	Preference 中有一个 onBindView(View view) 方法，重写该方法进行数据和视图的绑定。Preference中有两个处理 Preference 变化的监听器：OnPreferenceChangeListener 和 OnPreferenceChangeInternalListener，后者在内部使用。当调用notifyChanged通知变化时，调用的时内部的监听器，该监听器的实现在 PreferenceGroupAdapter 中：

```java
 public void onPreferenceChange(Preference preference) {
        notifyDataSetChanged();
    }
```

通知变更后会回调 Adapter 的 getView 方法，在这里再调用 Preference 的getView，而 getView 会调用 onBindView 方法，这样就实现了视图数据的刷新。如果需要通知用户的话，应该调用的时callChangeListener 方法，该方法调用的时机为用户改变了 Preference 但内部的状态未被设置的时候。

### Preference 子类分析

#### CheckBoxPreference

​	CheckBoxPreference 提供了一个 checkbox 的 widget，其父类是TwoStatePreference 。CheckBoxPreference 添加了两个自定义属性 summaryOn 和 summaryOff，分别表示开/关状态时summary的文本内容，然后在 onBindView 中初始化 CheckBox 状态和 summary。 



再看TwoStatePreference的源码：

​	首先看下点击事件部分：

```java
    @Override
    protected void onClick() {
        super.onClick();

        final boolean newValue = !isChecked();
        if (callChangeListener(newValue)) {//callChangeListener表示该新值是否应该保存
            setChecked(newValue);
        }
    }

    /**
     * Sets the checked state and saves it to the {@link SharedPreferences}.
     *
     * @param checked The checked state.
     */
    public void setChecked(boolean checked) {
        // Always persist/notify the first time; don't assume the field's default of false.
        final boolean changed = mChecked != checked;
        if (changed || !mCheckedSet) {
            mChecked = checked;
            mCheckedSet = true;//mCheckedSet 表示值已经已被设置过
            persistBoolean(checked);
            if (changed) {
                notifyDependencyChange(shouldDisableDependents());//通知依赖的Preference
                notifyChanged();//刷新视图
            }
        }
    }
```

​	TwoStatePreference 重写了默认值获取方法

```java
    @Override
    protected Object onGetDefaultValue(TypedArray a, int index) {
        return a.getBoolean(index, false);
    }
```

​	onSetInitialValue(...)方法用于初始化

```java
	@Override
    protected void onSetInitialValue(boolean restoreValue, Object defaultValue) {
        setChecked(restoreValue ? getPersistedBoolean(mChecked)
                : (Boolean) defaultValue);
    }	
	//onSetInitialValue 的第一个参数表示是否从SharedPreference 中恢复
```



#### DialogPreference

​	EditTextPreference、ListPreference等许多Preference都是基于对话框的Preference。

​	DialogPreference 增加的属性：

+ android:dialogTitle
+ android:dialogMessage
+ andorid:dialogIcon
+ android:positiveButtonText
+ android:negativeButtonText
+ android:dialogLayout



​	通过 setDialogLayoutResource(int) 方法可以设置对话框的content view布局。在点击后显示对话框：

```java
	@Override
    protected void onClick() {
        if (mDialog != null && mDialog.isShowing()) return;
        showDialog(null);//将创建一个AlertDialog
    }
```



##### EditTextPreference

​	EditTextPreference 是 DialogPreference 的一个子类，点击后将在对话框中显示一个EditText。其中两个构造函数：

```java
    public EditTextPreference(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes) {
        super(context, attrs, defStyleAttr, defStyleRes);     
        mEditText = new EditText(context, attrs);//创建EditText    
        // Give it an ID so it can be saved/restored
        mEditText.setId(com.android.internal.R.id.edit);
        /*
         * The preference framework and view framework both have an 'enabled'
         * attribute. Most likely, the 'enabled' specified in this XML is for
         * the preference framework, but it was also given to the view framework.
         * We reset the enabled state.
         */
        mEditText.setEnabled(true);
    }	
	public EditTextPreference(Context context, AttributeSet attrs) {
        this(context, attrs, com.android.internal.R.attr.editTextPreferenceStyle);
    }
```

​	该 style 在默认主题中定义了一个dialogLayout 属性，指向 preference_dialog_edittext.xml 布局文件

```java
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:layout_marginTop="48dp"
    android:layout_marginBottom="48dp"
    android:overScrollMode="ifContentScrolls">
    <LinearLayout
        android:id="@+android:id/edittext_container"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:padding="5dip"
        android:orientation="vertical">

        <TextView android:id="@+android:id/message"
            style="?android:attr/textAppearanceSmall"
            android:layout_marginBottom="48dp"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:textColor="?android:attr/textColorSecondary" />
    </LinearLayout>
</ScrollView>

```

​	在onBindDialogView(View)中进行数据和视图的绑定

```java
    @Override
    protected void onBindDialogView(View view) {//此处传入的 view 实际就是上面的布局表示的View
        super.onBindDialogView(view);
        EditText editText = mEditText;
        editText.setText(getText());    
        ViewParent oldParent = editText.getParent();
        if (oldParent != view) {
            if (oldParent != null) {
                ((ViewGroup) oldParent).removeView(editText);//移除旧的EditText
            }
            onAddEditTextToDialogView(view, editText);
        }
    }
```

​	EditTextPreference 同样重写了onSetInitialValue来初始化

```java
	@Override
    protected void onSetInitialValue(boolean restoreValue, Object defaultValue) {
        setText(restoreValue ? getPersistedString(mText) : (String) defaultValue);
    }
```









