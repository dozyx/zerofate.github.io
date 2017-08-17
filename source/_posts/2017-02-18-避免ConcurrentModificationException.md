---
title: 避免 ConcurrentModificationException
tags:
  - android
date: 2017-02-18 15:37:22
categories: 笔记
---

[Iterating through a Collection, avoiding ConcurrentModificationException when removing in loop](http://stackoverflow.com/questions/223918/iterating-through-a-collection-avoiding-concurrentmodificationexception-when-re)

**错误示范**

```java
for (Object i : l) {
    if (condition(i)) {
        l.remove(i);
    }
}
```



**正确姿势**

```java
List<String> list = new ArrayList<>();

// This is a clever way to create the iterator and call iterator.hasNext() like
// you would do in a while-loop. It would be the same as doing:
//     Iterator<String> iterator = list.iterator();
//     while (iterator.hasNext()) {
for (Iterator<String> iterator = list.iterator(); iterator.hasNext();) {
    String string = iterator.next();
    if (string.isEmpty()) {
        // Remove the current element from the iterator and the list.
        iterator.remove();
    }
}
```