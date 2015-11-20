---
date: 2013-09-26 20:30:00+00:00
layout: post
title:  "Activity向右Touch退出"
categories: Android
tags: Android 实例
excerpt: 在有返回操作的当前页面（activity）中，触摸此页面左右移动
---

在有返回操作的当前页面（activity）中，触摸此页面左右移动，而此页面会跟随手势左右移动，当向右移动超过一半的时候，就会继续播放完向右移动的动画（直到整个页面消失），然后就关闭此页面（activity）。

![img](/assets/2013-09-26-android-touch-back.png)

要实现这样的效果，需要两个要点：

<p></p>

1、设置当前的Activity的Window Background为透明的：
------

可以用java代码在Activity中这样设置：

{% highlight java linenos %}
    getWindow().setBackgroundDrawableResource(R.color.transparent);
{% endhighlight %}

也可以在xml文件中，创建一个style并且设置windowBackground属性为透明色，然后在mainfest文件中设置Activity的them属性中引用。

![img](/assets/2013-09-26-android-touch-back-2.png)

![img](/assets/2013-09-26-android-touch-back-3.png)

<p></p>

2、重写Activity的dispatchTouchEvent()方法：
------

dispatchTouchEvent()方法中调用了onTouchBack(ev)方法：

![img](/assets/2013-09-26-android-touch-back-4.png)

onTouchBack(ev)方法的实现是下面这样的，主要是调用了TouchBack类的onTouchBack()方法：

![img](/assets/2013-09-26-android-touch-back-5.png)

TouchBack类的onTouchBack()方法的实现是这样的：

![img](/assets/2013-09-26-android-touch-back-6.png)

在onDown()中，主要是做一些能否touch back的判断，以及做一些初始化的记录：

![img](/assets/2013-09-26-android-touch-back-7.png)

而checkMoveDirection(event)方法主要是判断是否达到touch back的条件，这个条件是：首先水平移动了一段距离；

也就是说如果首先向上或者向下移动了一段距离，那么就不会再满足touch back的条件。

这个很好说明，例如，如果在一个页面中有ListView，如果首先向上或者向下移动了一段距离，那么之后就一直是ListView的滚动事件，而不会执行touch back操作；相反，如果首先水平移动了一段距离，那么就会一直执行touch back操作而不会执行ListView的滚动事件了，直到手指离开屏幕。

其中判断Touch方向的一些方法已经封装在了Touch类里面。

![img](/assets/2013-09-26-android-touch-back-8.png)

真正移动这个页面的方法就是moveToX(float dx)方法，moveToX()方法通过设置view的padding从而达到view移动的效果。

![img](/assets/2013-09-26-android-touch-back-9.png)

最后，当手指离开这个页面的时候，就会调用smoothSwich()方法。在smoothSwich()方法中就是调用了前一篇文章中说到的自定义padding动画的那个Smooth类的一些方法进行移动动画的播放。

![img](/assets/2013-09-26-android-touch-back-10.png)

其中需要注意的是这些方法都属于TouchBack这个类的，而view属性是传入的Activity的DecorView。

![img](/assets/2013-09-26-android-touch-back-11.png)

实现Smooth中的ScroolToEndListener监听器，就可以在动画播放完之后做一些处理。

![img](/assets/2013-09-26-android-touch-back-12.png)

当页面向右退出后，就关闭Activity，当页面向左返回时，就设置View的背景为空。

![img](/assets/2013-09-26-android-touch-back-13.png)

另外，也需要实现页面的背景随着左右移动而改变透明度，其中在moveToX()方法中调用了Smooth.setPaddingLeft(view, (int) (paddingLeft + dx))方法，这个方法中就有随着padding的改变而改变透明度的一些设置：

![img](/assets/2013-09-26-android-touch-back-14.png)