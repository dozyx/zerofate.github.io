---
title: API Guides - DialogFragment
tags:
  - android
  - dialog
date: 2017-02-22 15:37:22
categories: 笔记
---

[DialogFragment](https://developer.android.com/reference/android/app/DialogFragment.html)



#### 用法

​	DialogFragment的一般用法有两种（以下代码摘自官网）：

1. 重载onCreateView方法，然后在Activity如下进行显示

   ```java
   void showDialog() {
       mStackLevel++;

       // DialogFragment.show() will take care of adding the fragment
       // in a transaction.  We also want to remove any currently showing
       // dialog, so make our own transaction and take care of that here.
       FragmentTransaction ft = getFragmentManager().beginTransaction();
       Fragment prev = getFragmentManager().findFragmentByTag("dialog");
       if (prev != null) {
           ft.remove(prev);
       }
       ft.addToBackStack(null);

       // Create and show the dialog.
       DialogFragment newFragment = MyDialogFragment.newInstance(mStackLevel);
       newFragment.show(ft, "dialog");
   }
   ```

2. 实现onCreateDialog(Bundle)来创建自定义的Dialog对象。这通常用来创建一个AlertDialog，然后通过以fragment的方式展示。

   ```java
       @Override
       public Dialog onCreateDialog(Bundle savedInstanceState) {
           int title = getArguments().getInt("title");

           return new AlertDialog.Builder(getActivity())
                   .setIcon(R.drawable.alert_dialog_icon)
                   .setTitle(title)
                   .setPositiveButton(R.string.alert_dialog_ok,
                       new DialogInterface.OnClickListener() {
                           public void onClick(DialogInterface dialog, int whichButton) {
                               ((FragmentAlertDialog)getActivity()).doPositiveClick();
                           }
                       }
                   )
                   .setNegativeButton(R.string.alert_dialog_cancel,
                       new DialogInterface.OnClickListener() {
                           public void onClick(DialogInterface dialog, int whichButton) {
                               ((FragmentAlertDialog)getActivity()).doNegativeClick();
                           }
                       }
                   )
                   .create();
       }
   ```

   activity显示

   ```java
   void showDialog() {
       DialogFragment newFragment = MyAlertDialogFragment.newInstance(
               R.string.alert_dialog_two_buttons_title);
       newFragment.show(getFragmentManager(), "dialog");
   }
   ```

   > 此处未将fragment放入back stack，但dialog通常是模态的，所以仍会进行back stack的操作。



#### Selecting Between Dialog or Embedding

​	DialogFragment也可以选择作为普通的fragment来使用。通常可以根据使用fragment的方式自动选择作为对话框还是普通fragment，但也可以使用[setShowsDialog(boolean)](https://developer.android.com/reference/android/app/DialogFragment.html#setShowsDialog(boolean))来自定义。

> This is normally set for you based on whether the fragment is associated with a container view ID passed to FragmentTransaction.add(int, Fragment). If the fragment was added with a container, setShowsDialog will be initialized to false; otherwise, it will be true.

如：	

```java
public static class MyDialogFragment extends DialogFragment {
    static MyDialogFragment newInstance() {
        return new MyDialogFragment();
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
            Bundle savedInstanceState) {
        View v = inflater.inflate(R.layout.hello_world, container, false);
        View tv = v.findViewById(R.id.text);
        ((TextView)tv).setText("This is an instance of MyDialogFragment");
        return v;
    }
}
```

​	显示为对话框

```java
void showDialog() {
    // Create the fragment and show it as a dialog.
    DialogFragment newFragment = MyDialogFragment.newInstance();
    newFragment.show(getFragmentManager(), "dialog");
}
```

​	作为内容显示

```java
FragmentTransaction ft = getFragmentManager().beginTransaction();
DialogFragment newFragment = MyDialogFragment.newInstance();
ft.add(R.id.embedded, newFragment);
ft.commit();
```