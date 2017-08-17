---
title: 双屏功能 - Presentation
tags:
  - android
  - 双屏
date: 2017-04-06 15:37:22
categories: 笔记
---

java.lang.Object
   ↳	android.app.Dialog
 	   ↳	android.app.Presentation



​	Presentation 是用于显示功能（presentation，展示、显示）的基类，**是一种特殊类型的 dialog** ，其作用是将内容显示到第二个显示屏上。Presentation 在创建时会关联到目标显示屏，并根据显示屏参数配置它的 context 和资源。

​	特别注意，**presentation 的 Context 与包含它的 Activity 的 context 是不一样的**。为确保能够正确加载目标显示屏的大小和密度，使用presentation 自己的 context 来inflate一个 presentation 布局和加载其他资源十分重要。

​	当关联的显示屏被移除时，presentation 会自动被 cancel。在activity 处于paused 或 resumed状态时，activity 应当维护 presentation 中所有内容的 pause 和 resume。



## 使用

### 选择 presentation 的显示屏

​	在显示 presentation 之前，需要选择用于显示的显示屏。

​	有两种选择显示屏的主要方式：

+ 使用 media router 来选择 presentation 显示屏
+ 使用 display manager 来选择 presentation 显示屏



#### 使用 media router 来选择 presentation 显示屏

> router：路由器
>
> route：路线
>
> presentation display：可以显示 presentation 的显示屏，通常指第二屏幕

​	**使用 MediaRouter 是选择presentation 显示屏最简单的方式**，媒体路由器（media route）服务会跟踪系统中哪个 audio 和 video 路线（route）是可用的。当 route 被选择、取消选择或者 route 的偏好 presentation 显示屏改变时，媒体路由器都会发出通知。因此，应用可以通过简单地关注这些通知，自动在偏好的 presentation 显示屏上 show 或 dismiss 一个presentation。 

​	偏好的 presentation 显示屏是 media router 推荐的用于在第二显示屏上显示内容的显示屏。	有时候，可能不只一个偏好 presentation 显示屏，这种情况会出现在应用不使用 presentation 来显示本地的内容时。



示例：

使用media router 创建并在偏好 presentation 显示屏（使用 getPresentationDisplay() 获得）上显示一个 presentation。

```java
MediaRouter mediaRouter = (MediaRouter) context.getSystemService(Context.MEDIA_ROUTER_SERVICE);
 MediaRouter.RouteInfo route = mediaRouter.getSelectedRoute();
 if (route != null) {
     Display presentationDisplay = route.getPresentationDisplay();
     if (presentationDisplay != null) {
         Presentation presentation = new MyPresentation(context, presentationDisplay);
         presentation.show();
     }
 }
```

ApiDemo中的[PresentationWithMediaRouterActivity示例](https://github.com/android/platform_development/blob/master/samples/ApiDemos/src/com/example/android/apis/app/PresentationWithMediaRouterActivity.java)展示了如何使用 media router 自动切换主activity 显示内容和当一个 presentation 显示屏可用时在presentation中显示内容。



#### 使用 display manager 来选择 presentation 显示屏

​	display manager服务提供了方法来罗列和描述所有与系统关联的显示屏，包括可能用于 presentation 的显示屏。

​	display manager 会跟踪系统中所有的显示屏，但是并不是所有的显示屏都适合用来显示 presentation。如，一个activity 可能会尝试在主显示屏上显示一个 presentation，这将导致其自身内容变得模糊（相当于在activity上面显示了一个dialog）。



示例：

使用 getDisplay(String) 和 DISPLAY_CATEGORY_PRESENTATION 类别来标识合适的显示屏。

```java
 DisplayManager displayManager = (DisplayManager) context.getSystemService(Context.DISPLAY_SERVICE);
 Display[] presentationDisplays = displayManager.getDisplays(DisplayManager.DISPLAY_CATEGORY_PRESENTATION);
 if (presentationDisplays.length > 0) {
     // If there is more than one suitable presentation display, then we could consider
     // giving the user a choice.  For this example, we simply choose the first display
     // which is the one the system recommends as the preferred presentation display.
     Display display = presentationDisplays[0];
     Presentation presentation = new MyPresentation(context, presentationDisplay);
     presentation.show();
 }
```

ApiDemo 中的[PresentationActivity 示例](https://github.com/android/platform_development/blob/master/samples/ApiDemos/src/com/example/android/apis/app/PresentationActivity.java)展示了如何使用display manager 罗列显示屏，并同时在多个 presentation屏幕上显示内容。









