---
date: 2013-10-18 20:30:00+00:00
layout: post
title:  "点击编辑框外任意点，隐藏软键盘"
categories: Android
tags: Android 实例
excerpt: 重写dispatchTouchEvent()方法
---

重写dispatchTouchEvent()方法：

![img](/assets/2013-10-18-android-edit-hide.png)

其中editTouchEvent()方法的实现是：

![img](/assets/2013-10-18-android-edit-hide-2.png)

通过down事件和move事件判断是否符合点击事件，其中move事件中如果移动的距离小于DY被认为是点击事件。在up方法中调用方法判断被点击的是否是一个EditText，如果不是，则以藏软键盘。

![img](/assets/2013-10-18-android-edit-hide-3.png)

其中checkEditing()方法中调用了InputUtil.isEditing(getWindow().getDecorView(), event)方法，isEditing()方法会在DecorView中去寻找被点击的那个控件，并且判断这个控件是否为EditText：

![img](/assets/2013-10-18-android-edit-hide-4.png)