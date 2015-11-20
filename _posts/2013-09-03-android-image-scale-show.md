---
date: 2013-09-03 20:30:00+00:00
layout: post
title:  "屏幕的某位置弹出缩放动画"
categories: Android
tags: Android 实例
excerpt: 点击下图的一个ImageView，然后就从这个ImageView的位置弹出一个渐渐放大的对话框
---

这种效果是这样的：点击下图的一个ImageView，然后就从这个ImageView的位置弹出一个渐渐放大的对话框，这个对话框中也包括一个ImageView，在动画的过程中，这个ImageView的初始大小与图中的这个被点击的ImageView的大小是一样的，而最终的大小是占满屏幕。

![img](/assets/2013-09-03-android-image-scale-show.png)

首先，获取被点击的ImageView的位置：

![img](/assets/2013-09-03-android-image-scale-show-2.png)
![img](/assets/2013-09-03-android-image-scale-show-3.png)

然后通过Rect去创建ScaleAnimation动画对象；最后通过调用view.startAnimation()方法播放动画。

![img](/assets/2013-09-03-android-image-scale-show-4.png)