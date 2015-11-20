---
date: 2013-05-08 20:30:00+00:00
layout: post
title:  "圆角"
categories: Android
tags: Android 实例
excerpt: 使用xml文件定义圆角，canvas.drawBitmap()，canvas.clipPath()
---

方式一：使用xml文件定义圆角
------
![img](/assets/2013-05-08-android-round-corner.png)

<p></p>

方式二：canvas.drawBitmap()
------
此方法需要new一个canvas来生成圆角。

![img](/assets/2013-05-08-android-round-corner-2.png)

<p></p>

方式三：canvas.clipPath()
------

![img](/assets/2013-05-08-android-round-corner-3.png)

![img](/assets/2013-05-08-android-round-corner-4.png)

通过第三种方法的效果作用范围更广，使用更灵活。

<p></p>

