---
date: 2013-11-08 20:30:00+00:00
layout: post
title:  "Android 界面性能优化方案"
categories: Android
tags: Android 实例 性能优化
excerpt: 优化原则：不要堵塞UI线程，开启新的线程去做复杂处理，在UI线程更新界面；
---

优化的原因
------
> * 用户体验
* 用户体验
* 用户体验
* 用户体验
* ……
<p></p>

几个概念
------

* 线程安全：非UI线程不能更新UI组件
* [Android的进程与线程（3）线程安全问题](http://blog.csdn.net/yaolingrui/article/details/7420074)
* 帧率
<p></p>

优化原则
------

* 不要堵塞UI线程
* 开启新的线程去做复杂处理，在UI线程更新界面；
![img](/assets/2013-11-08-android-performance.png)

* getView()、onDraw()、scroll类的方法等运行要简洁快速
![img](/assets/2013-11-08-android-performance-2.png)

* 更简单地布局：少嵌套、适当使用权重布局和相对布局
![img](/assets/2013-11-08-android-performance-3.png)

* Adapter优化：重用View和避免findViewById()
![img](/assets/2013-11-08-android-performance-4.png)

* 避免显示的布局里重复使用同一个背景
![img](/assets/2013-11-08-android-performance-5.png)
![img](/assets/2013-11-08-android-performance-6.png)

* 不显示的View，就隐藏不显示
* 在有些情况下Activity的背景可以设置为空
> getWindow().setBackgroundDrawable (null); 
android:windowBackground="@null" 
![img](/assets/2013-11-08-android-performance-7.png)

* 纯色的背景比图片背景更高效
![img](/assets/2013-11-08-android-performance-8.png)

* 美术资源要求：

> 1. 纯色的背景，只需提供背景的颜色值；
2. 纹理背景尽量提供尽量小的一小块图片，而开发人员会根据这个
3. 可以拉伸的背景，尽量提供尽量小的一小块图片，而开发人员会根据这个小图片拉伸背景。 
4. 需要进行矢量拉伸的图片要做.9.png格式处理。

还可以参考以下的文章：

* [android UI性能优化](http://blog.csdn.net/androidzhaoxiaogang/article/details/8673654)
* [Android UI 优化](http://wenku.baidu.com/view/02e26b4ce518964bcf847c04.html)