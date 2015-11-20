---
date: 2013-05-01 20:30:00+00:00
layout: post
title:  "Android Library"
categories: Android
tags: Android 学习
excerpt: Library工程的好处，一些util 方法或者我们自定义的控件放到Library工程，实现复用
---

1.Library工程的好处
------
> * 一些util 方法或者我们自定义的控件放到Library工程，实现复用；
* 相比jar 包而言，他可以实现资源文件的复用甚至覆盖；
* 模块化设计实现代码共享，便于管理，提高效率；

2.Library工程的设置
------
在包资源管理窗口，右键点击要进行设置的项目->属性->Android->勾选 Is Libarary；

![img](/assets/2013-05-01-android-library.png)

<p></p>

3.Library工程的引用
------
在包资源管理窗口，右键点击要引用Library工程的项目->属性->Android->右下Add ->选择要引用的Libarary->确定；

![img](/assets/2013-05-01-android-library-2.png)

![img](/assets/2013-05-01-android-library-3.png)

<p></p>

