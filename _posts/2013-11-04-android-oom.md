---
date: 2013-11-04 20:30:00+00:00
layout: post
title:  "Android OOM的解决方案"
categories: Android
tags: Android 实例 OOM
excerpt: 在Android中，一个Process 只能使用16M内存，如果超过了这个限制就会跳出这个异常；
---

造成OOM的原因
------
![img](/assets/2013-11-04-android-oom.png)
-----P103

* 在Android中，一个Process 只能使用16M内存，如果超过了这个限制就会跳出这个异常；
* 内存不够用或者耗尽了
* [Android Out Of Memory(OOM) 的详细研究](http://www.cnblogs.com/wanqieddy/archive/2012/07/18/2597471.html)
* 主要原因

> 图片过大
图片过多
页面过多
内存泄漏

解决的终极原则：让更少的Bitmap驻留在内存
------

* 二级缓存：
![img](/assets/2013-11-04-android-oom-2.png)
![img](/assets/2013-11-04-android-oom-3.png)

* 图片做软、弱引用
![img](/assets/2013-11-04-android-oom-4.png)

相关文章-强、软、弱、和虚: 
[1](http://zhangjunhd.blog.51cto.com/113473/53092/)
[2](http://mobile.51cto.com/abased-406998.htm)

* 加载缩小的图片
![img](/assets/2013-11-04-android-oom-5.png)

* 动态释放内存
![img](/assets/2013-11-04-android-oom-6.png)

> * 内存管理尽量交由系统自动管理
* trying to use a recycled bitmap错误
* [Android系统中Bitmap是否有调用recycle方法的必要性](http://www.oschina.net/question/565065_67273)

* 裁剪背景图、弱引用背景图
![img](/assets/2013-11-04-android-oom-7.png)

* 释放页面的内容
![img](/assets/2013-11-04-android-oom-8.png)
![img](/assets/2013-11-04-android-oom-9.png)

* 设置堆内存的相关参数
![img](/assets/2013-11-04-android-oom-10.png)
[关于Android堆内存的设置](http://www.cnblogs.com/jacktu/archive/2010/12/30/1921475.html)

* 避免内存泄漏

> * [Android内存泄漏的各种原因详解](http://mobile.51cto.com/abased-406286.htm)
* 长期持有了一个Context的引用
* [如何避免Android内存泄漏——Context](http://www.cnblogs.com/shaweng/archive/2012/06/29/2570413.html)
* 垃圾回收器不能处理内存泄漏
* [使用内存分析工具：MAT](http://wenku.baidu.com/view/0bffafff700abb68a882fb04.html)