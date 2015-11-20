---
date: 2013-09-05 20:30:00+00:00
layout: post
title:  "列表项的删除动画效果"
categories: Android
tags: Android 实例
excerpt: 被删除的那一项渐渐的消失（或者向左或右移除），而其他项向被删除的那一项慢慢靠拢
---

删除动画的效果是这样的：被删除的那一项渐渐的消失（或者向左或右移除），而其他项向被删除的那一项慢慢靠拢；

首先，被删除的那一项渐渐的消失（或者向左或右移除）的动画是：

![img](/assets/2013-09-05-android-list-remove-anim.png)

其他项向被删除的那一项慢慢靠拢，是通过一帧一帧地减小这个被删除的View的高度实现的：

![img](/assets/2013-09-05-android-list-remove-anim-2.png)

这个动画需要实现Runnable接口，重写run方法，并在这个run方法中调用onUpdate()方法，而其中mListView.post(this)方法使得这个动画继续播放下一帧：

![img](/assets/2013-09-05-android-list-remove-anim-3.png)

启动动画，调用mListView.post(this)方法：

![img](/assets/2013-09-05-android-list-remove-anim-4.png)

当然，为了使这个动画的内容丰富，我们应该要实现cancel()、stop()这些以及添加一些监听器等：

![img](/assets/2013-09-05-android-list-remove-anim-5.png)