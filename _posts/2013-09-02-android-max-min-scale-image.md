---
date: 2013-09-02 20:30:00+00:00
layout: post
title:  "有最大最小宽高值的ImageView"
categories: Android
tags: Android 实例
excerpt: 以下ImageView显示一张图片，ImageView的大小根据图片的大小进行设置
---

以下ImageView显示一张图片，ImageView的大小根据图片的大小进行设置；其中ImageView的宽和高都有一个最大最小值；最大时是左图上面的ImageView的宽度和高度；最小时是右图上面的的ImageView的宽度。

![img](/assets/2013-09-02-android-max-min-scale-image.png)

我们可以通过下面的算法对ImageView的width和height进行设置，就可以达到想要的效果了。

其中imageView.setScaleType(ScaleType.CENTER_CROP)可以使得imageView截取图片的中间部分进行显示。例如图片很长的时候，就可以截取这张图片的中间爱你部分来显示。

![img](/assets/2013-09-02-android-max-min-scale-image-2.png)