---
date: 2013-08-16 20:30:00+00:00
layout: post
title:  "改变图片的颜色"
categories: Android
tags: Android 实例
excerpt: 下图动态条目的声音图标，原来是白色的，可以通过代码改变图标为红色
---

下图动态条目的声音图标，原来是白色的，可以通过代码改变图标为红色。

![img](/assets/2013-08-16-android-icon-color.png)

代码如下，原理大概是：把这张资源图片里面的所有透明部分提取出来，然后重新绘制一张新的Bitmap，透明的部分继续绘制成透明的，非透明的部分就绘制成红色（或者你想要的颜色）。所以，使用这种方法的一个条件是这张图片带有透明的部分；

![img](/assets/2013-08-16-android-icon-color-2.png)


