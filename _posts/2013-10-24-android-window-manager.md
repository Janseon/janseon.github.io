---
date: 2013-10-24 20:30:00+00:00
layout: post
title:  "Activity的WindowManager"
categories: Android
tags: Android 实例 WindowManager
excerpt: 在Activity中有几个setContentView()方法，可以设置Activity的页面内容
---

使用Activity的WindowManager可以生成真正浮动的View，这种浮动就像是Toast的那种弹出浮动。我们可以根据需要自己进行自定义，一个用法例子如下，这个例子是自定义Toast的一段代码：

![img](/assets/2013-10-24-android-window-manager.png)

同样需要设置布局参数、拥有addView()、removeView()方法