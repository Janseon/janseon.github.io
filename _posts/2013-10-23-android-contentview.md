---
date: 2013-10-23 20:30:00+00:00
layout: post
title:  "Activity的ContentView添加删除"
categories: Android
tags: Android 实例 ContentView
excerpt: 在Activity中有几个setContentView()方法，可以设置Activity的页面内容
---

在Activity中有几个setContentView()方法，可以设置Activity的页面内容，另外还有一个addContentView()方法，也可以设置Activity的页面内容。

但两者不一样的地方是的。

setContentView()方法方法会先清空以前设置的页面的所有内容:

![img](/assets/2013-10-23-android-contentview.png)

PhoneWindow中的setContentView()方法如下：

![img](/assets/2013-10-23-android-contentview-2.png)

而addContentView()方法不会清空以前设置的页面的所有内容，并且在原有的页面内容上添加ContentView：

![img](/assets/2013-10-23-android-contentview-3.png)

PhoneWindow中的addContentView()方法如下：

![img](/assets/2013-10-23-android-contentview-4.png)

于是乎，我们就可以使用addContentView()方法大作文章：

####1、添加标题栏

![img](/assets/2013-10-23-android-contentview-5.png)

其中contentView就是想要设置的页面内容，这个要需要先设置topMargin的大小与标题栏的高度一致；titleBar就是要设置的标题栏的内容；需要注意的是要先添加contentView，再添加titleBar，因为titleBar要覆盖在contentView上面。效果图如下，红色框那部分是titleBar，其余的是contentView：

![img](/assets/2013-10-23-android-contentview-6.png)

####2、添加底部控制栏

添加底部的控制栏与添加标题栏的原理是一样的，只是设置的gravity不一样：

![img](/assets/2013-10-23-android-contentview-7.png)

####3、添加浮动控件

你可以在任意位置添加一个控件或布局

![img](/assets/2013-10-23-android-contentview-8.png)

####4、移除添加的contentView

![img](/assets/2013-10-23-android-contentview-9.png)

####5、模拟弹出对话框

调用setFilledView()方法，然后再对contentView设置一些弹出动画

![img](/assets/2013-10-23-android-contentview-10.png)

至于对话框的消失，可以调用4中的removeView()方法，然后设置一些动画就可以了。