---
date: 2012-12-22 20:30:00+00:00
layout: post
title:  "加载xml布局文件原理"
categories: Android SDK源码
tags: Android LayoutInflater
excerpt: 我们以LayoutInflater类的public View inflate(int resource, ViewGroup root)方法为例
---

我们以LayoutInflater类的public View inflate(int resource, ViewGroup root)方法为例；当调用inflate()方法的时候，是对一个xml文件进行加载的，我们可以跟踪这个方法：

inflate(int resource, ViewGroup root)方法会调用同个类里面的inflate(XmlPullParser parser, ViewGroup root, boolean attachToRoot)方法，这个方法就是一个很关键的方法，我们可以查看这个方法的代码：

{% highlight java linenos %}
    public View inflate(XmlPullParser parser, ViewGroup root, boolean attachToRoot) {
        synchronized (mConstructorArgs) {
            final AttributeSet attrs = Xml.asAttributeSet(parser);
            mConstructorArgs[0] = mContext;
            View result = root;

            try {
                //……
                final String name = parser.getName();
                //……
                if (TAG_MERGE.equals(name)) {
                    //……
                } else {
                    // Temp is the root view that was found in the xml
                    View temp = createViewFromTag(name, attrs);

                    ViewGroup.LayoutParams params = null;

                    if (root != null) {
                        if (DEBUG) {
                            System.out.println("Creating params from root: " +
                                    root);
                        }
                        // Create layout params that match root, if supplied
                        params = root.generateLayoutParams(attrs);
                        if (!attachToRoot) {
                            // Set the layout params for temp if we are not
                            // attaching. (If we are, we use addView, below)
                            temp.setLayoutParams(params);
                        }
                    }

                    if (DEBUG) {
                        System.out.println("-----> start inflating children");
                    }
                    // Inflate all children under temp
                    rInflate(parser, temp, attrs);
                    if (DEBUG) {
                        System.out.println("-----> done inflating children");
                    }

                    // We are supposed to attach all the views we found (int temp)
                    // to root. Do that now.
                    if (root != null && attachToRoot) {
                        root.addView(temp, params);
                    }

                    // Decide whether to return the root that was passed in or the
                    // top view found in xml.
                    if (root == null || !attachToRoot) {
                        result = temp;
                    }
                }

            } catch (XmlPullParserException e) {
                //……
            }

            return result;
        }
    }
{% endhighlight %}

这个inflate()方法会使用XmlPullParser对象来对xml文件进行解析，先不管merge相关的布局，其解析的过程：

记录xml的所有属性attrs = Xml.asAttributeSet(parser)；

解析最外层的layout View的name，这个name标志了这个View的类型，例如LinearLayout、RelativeLayout、以及其他类型的View等；

调用temp = createViewFromTag(name, attrs)，创建一个对应类型的View。

如果temp 存在parent，也就是root，则调用params = root.generateLayoutParams(attrs)来创建相应的依附于root的参数对象，创建这个参数对象的时候，也同事把相应的attrs属性传进去，这些属性就是从xml文件里面解析出来的。

然后调用rInflate(parser, temp, attrs)方法，继续往下递归地解析；

最后，如果temp存在parent，也就是root，再调用root.addView(temp, params)把这个View添加进root。

我们再看看rInflate(parser, temp, attrs)方法的实现，这个实现里面是使用递归的方法进行解析的，看看代码：

{% highlight java linenos %}
    private void rInflate(XmlPullParser parser, View parent, final AttributeSet attrs)
            throws XmlPullParserException, IOException {

        final int depth = parser.getDepth();
        int type;

        while (((type = parser.next()) != XmlPullParser.END_TAG ||
                parser.getDepth() > depth) && type != XmlPullParser.END_DOCUMENT) {

            if (type != XmlPullParser.START_TAG) {
                continue;
            }

            final String name = parser.getName();
            
            if (TAG_REQUEST_FOCUS.equals(name)) {
                parseRequestFocus(parser, parent);
            } else if (TAG_INCLUDE.equals(name)) {
                //……
            } else if (TAG_MERGE.equals(name)) {
                throw new InflateException("<merge /> must be the root element");
            } else {
                final View view = createViewFromTag(name, attrs);
                final ViewGroup viewGroup = (ViewGroup) parent;
                final ViewGroup.LayoutParams params = viewGroup.generateLayoutParams(attrs);
                rInflate(parser, view, attrs);
                viewGroup.addView(view, params);
            }
        }

        parent.onFinishInflate();
    }
{% endhighlight %}

rInflate(parser, temp, attrs)方法的实现和inflate()方法的实现是相似的，其过程也需要：

> * parser.getName()；
* createViewFromTag(name, attrs)；
* generateLayoutParams(attrs)；
* rInflate(parser, view, attrs)；
* addView(view, params)。


其中比较重要的是rInflate(parser, temp, attrs)方法里面继续调用自己进行递归。

另外就是Activity类的setContentView(int layoutResID)方法：

{% highlight java linenos %}
    public void setContentView(int layoutResID) {
        getWindow().setContentView(layoutResID);
    }
{% endhighlight %}

这里有一个疑问，getWindow()返回的是什么东东呢？getWindow()返回一个Window类型的对象，但是Window是一个抽象类，并不能直接实例化一个对象。那么getWindow()返回的肯定是Window类的子类对象。那么这个子类是什么呢？为了得到这个类对象到底是什么，我们可以做一个实验，那就是写一个空内容的xml文件取名为test.xml，如下：

{% highlight xml linenos %}
    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout > </LinearLayout>
{% endhighlight %}

然后在一个Activity的onCreate()方法里面调用setContentView(R.layout.test)方法来加载这个layout的xml文件。

程序会运行错误，有如下错误：
![img](/assets/2012-12-22-android-layout-loading.png)

从上面的错误可以看出来，原来Activity的setContentView(int layoutResID)方法是调用了PhoneWindow类的setContentView(int layoutResID)方法，那么现在我们打开PhoneWindow类文件进行研究。如果在eclipse上面打不开这个PhoneWindow类文件，那么就需要在SDK的源代码的目录下去寻找PhoneWindow.java文件来阅读。这个SDK的源代码的目录是自己人为的添加进去的，如果不清楚这一点，那么就需要看看有“阅读SDK源代码”内容的那一章节。

打开PhoneWindow类文件后，点击ctrl+F寻找setContentView(int layoutResID)方法，代码如下：

{% highlight java linenos %}
 	public void setContentView(int layoutResID) {
        if (mContentParent == null) {
            installDecor();
        } else {
            mContentParent.removeAllViews();
        }
        mLayoutInflater.inflate(layoutResID, mContentParent);
        final Callback cb = getCallback();
        if (cb != null) {
            cb.onContentChanged();
        }
    }
{% endhighlight %}
    
原来这个setContentView()方法，最终还是调用了LayoutInflater类的inflate(int resource, ViewGroup root)方法。其中mContentParent参数是这个PhoneWindow里面本来就存在的一个FrameLayout。如果mContentParent为空的话，就会先初始化一个FrameLayout赋值给mContentParent。而mContentParent还有parent，这个parent就是一个DecorView，叫做装饰View。Android系统的要显示一个屏幕的内容，这些内容是有一定的层次关系的，详细可以查看下面的文章：

* [学习Android界面设计的超级利器HierarchyView](http://www.doc88.com/p-296361022788.html)

上面使用了HierarchyView工具来查看PhoneWindow里面的布局结构。