---
title: Serializable 和 Parcelable 序列化
tags:
  - java
date: 2017-05-09 15:37:22
categories: 笔记
---

​	Serializable 和 Parcelable是序列化的两种方式，对象序列化后，可用于进程间的对象传递，甚至在网络上传递。Serialzable 是一个标记接口，没有任何的方法或字段；Parcelable 同样也是一个接口，通过实现其方法和一个 CREATOR 对象来达到序列化和反序列化目的。Serialzable 会将整个对象序列化，而 Parcelable 可以自行选择需要序列化保存的部分，所以 Parcelable 具有更好的效率。



### Serializable

```java
public interface Serializable {
}
```

在序列化和反序列化过程中，如果需要特殊处理，可以在类中实现以下的特殊方法：

`private void writeObject(java.io.ObjectOutputStream out) throws IOException`



`private void readObject(java.io.ObjectInputStream in) throws IOException, ClassNotFoundException;`



`private void readObjectNoData() throws ObjectStreamException; `



### Parcelable

​	Parcelable 接口使对象可以转为 [Parcel](https://developer.android.com/reference/android/os/Parcel.html) 或者从[Parcel](https://developer.android.com/reference/android/os/Parcel.html) 中恢复。实现 Parcelable 接口的类还必须拥有一个继承`Parcelable.Creator`接口的非空静态字段 `CREATOR`。

​	典型的 Parcelable 实现：

```java
public class MyParcelable implements Parcelable {
     private int mData;

     public int describeContents() {
         return 0;
       //通常返回0即可。其他情形，如 Parcelable 对象需要利用到文件描述符，则会返回CONTENTS_FILE_DESCRIPTOR。
     }

     public void writeToParcel(Parcel out, int flags) {
         out.writeInt(mData);
       //使用 Parcel 对象的writeXX方法存入数据
     }

     public static final Parcelable.Creator<MyParcelable> CREATOR
             = new Parcelable.Creator<MyParcelable>() {
         public MyParcelable createFromParcel(Parcel in) {
             return new MyParcelable(in);
         }

         public MyParcelable[] newArray(int size) {
             return new MyParcelable[size];
         }
     };
     
     private MyParcelable(Parcel in) {
         mData = in.readInt();
       //注意：在取出数据时的顺序必须与写入顺序一致
     }
 }

```















