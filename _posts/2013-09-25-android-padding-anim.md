---
date: 2013-09-25 20:30:00+00:00
layout: post
title:  "自定义padding动画"
categories: Android
tags: Android 实例
excerpt: 通过一帧一帧地设置一个View的padding的大小来实现view的移动或者隐藏动画
---

通过一帧一帧地设置一个View的padding的大小来实现view的移动或者隐藏动画。

在一些可下拉刷新的控件中，下拉刷新的后，header view可以使用padding动画来实现缓慢返回和隐藏的动画效果。

首先定义一个Smooth类：

![img](/assets/2013-09-25-android-padding-anim.png)

开始执行向上返回动画的方法，其中调用了handler.post(scroolToTopRunnable)，开始执行第一帧：

![img](/assets/2013-09-25-android-padding-anim-2.png)

scroolToTopRunnable的实现如下，其中run方法调用了：scroolToTop()方法：

![img](/assets/2013-09-25-android-padding-anim-3.png)

scroolToTop()方法根据当前的padding和最终要设置的padding，计算出这一帧要设置的padding，然后设置这一帧的padding，如果设置的这一帧的padding和最终要设置的padding是相等的，那么就播放完动画，否则继续调用了handler.post(scroolToTopRunnable)，开始播放下一帧：

![img](/assets/2013-09-25-android-padding-anim-4.png)

同理的，我们可以定义向左、向右、向下的一些动画：

![img](/assets/2013-09-25-android-padding-anim-5.png)

![img](/assets/2013-09-25-android-padding-anim-6.png)

并且，我们还可以添加动画的监听器，如上代码：定义了ScroolToEndListener接口，等到动画播放完之后就调用接口的onScroolToEnd()方法；我们在外面实现这样的接口，就可以在动画播放完之后做一些处理了。