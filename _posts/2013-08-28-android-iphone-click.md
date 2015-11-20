---
date: 2013-08-28 20:30:00+00:00
layout: post
title:  "仿iphone的点击效果"
categories: Android
tags: Android 实例
excerpt: 以下的返回按钮，没有点击的情况，是一个简单的返回图标，点击这个按钮时
---

以下的返回按钮，没有点击的情况，是一个简单的返回图标，点击这个按钮时，这个按钮的底部会显示一片渐进的白色；

![img](/assets/2013-08-28-android-iphone-click.png)

这种效果可以使用ImageView来实现，ImageView的前景固定为一个返回的图标，背景是一个selector，这个selector正常状态是透明背景，点击状态是渐进白色的背景

![img](/assets/2013-08-28-android-iphone-click-2.png)

![img](/assets/2013-08-28-android-iphone-click-3.png)

这个ImageView的高占满标题栏的高度，而宽比高还要大；而前景的返回图标的显示大小，可以通过设置padding的大小来适应；并且这样子，也可以把返回按钮的点击范围扩大，使得点击更加灵敏；

所以，有时候我们想扩大按钮的点击范围，可以考虑使用ImageView来当Button使用.

