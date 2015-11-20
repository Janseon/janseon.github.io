---
date: 2013-03-25 20:30:00+00:00
layout: post
title:  "Android学习总结"
categories: Android Summary
tags: Android 学习 总结
excerpt: 建立自己的代码库、开源项目、反编译、查看SDK源代码
---

1.建立自己的代码库
------
即学即用，即收藏。如以下目录：

![img](/assets/2013-03-25-android-learn-summary.png)

<p></p>

2.开源项目
------
* [Android开源项目](http://www.open-open.com/75.htm)
* [Android开源项目源码下载](http://www.cnblogs.com/salam/archive/2010/10/26/1861779.html)
* [10个经典的Android开源项目](http://www.eoeandroid.com/code/2012/0321/978.html)
* [共有584款 Android开源软件](http://www.oschina.net/project/tag/189/android?lang=0&os=189&sort=time&p=4)

<p></p>

3.反编译
------
反编译的三个工具：ApkTool-2xml、dex2jar、jd-gui。

* [Android APK反编译详解](http://blog.csdn.net/sunboy_2050/article/details/6727581)
* [android反编译和防止反编译的方法](http://tech.it168.com/a2012/0201/1305/000001305453.shtml)

<p></p>

4.查看SDK源代码
------
把本文件夹里面，Android开发文档\src\目录下的sources_2.2.zip压缩文件解压到sources目录下，解压后sources目录里面是这样的：

![img](/assets/2013-03-25-android-learn-summary-2.png)

然后把sources目录整个目录拷贝到android-sdk的那个目录下，具体的目录是android-sdk-windows\platforms\android-8\，如下：

![img](/assets/2013-03-25-android-learn-summary-3.png)

然后重启eclipse，就可以通过ctrl+点击某一个类，而跳进SDK的源代码去了。

<p></p>

5.应用的开发框架
------
查看《Android开发规则》的文档

<p></p>

6.Activity原理
------
就是所谓的高级内容：JNI、，openGL ES控件编程、Android移植编程，这些内容不是必须的，不讲了。
<p></p>

7.应用运行机制、事件分发机制（Key、Touch）
------
略
<p></p>

8.ViewRoot
------
* [android的窗口机制分析------ViewRoot类](http://blog.csdn.net/windskier/article/details/6957901)
* [Android对Handler和ViewRoot的理解](http://hi.baidu.com/gaogaf/item/3f5008f06b742910d6ff8c49)
* [Android 窗口管理](http://www.blogjava.net/lihao336/archive/2010/11/24/338962.html)
* [android中View, Window, Activity, WindowManager，ViewRoot几者之间的关系 ](http://chriszeng87.iteye.com/blog/1096633)

<p></p>


