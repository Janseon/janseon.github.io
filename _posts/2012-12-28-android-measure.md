---
date: 2012-12-28 20:30:00+00:00
layout: post
title:  "View的测量和重绘"
categories: Android SDK源码
tags: Android Measrue Invalidate Draw
excerpt: 首先，我们要了解android系统是怎样加载一个xml的layout文件，这个可以参考前面的章节
---

首先，我们要了解android系统是怎样加载一个xml的layout文件，这个可以参考前面的章节：“加载xml布局文件原理”的内容。了解了这些内容之后，就可以继续研究了。

在inflate()方法和rInflate(parser, temp, attrs)方法中都会调用createViewFromTag(name, attrs)方法来创建新的View，我们可以看看这个创建View的方法的实现：

{% highlight java linenos %}
    View createViewFromTag(String name, AttributeSet attrs) {
        if (name.equals("view")) {
            name = attrs.getAttributeValue(null, "class");
        }

        if (DEBUG) System.out.println("******** Creating view: " + name);

        try {
            View view = (mFactory == null) ? null : mFactory.onCreateView(name,
                    mContext, attrs);

            if (view == null) {
                if (-1 == name.indexOf('.')) {	
                    view = onCreateView(name, attrs);
                } else {
                    view = createView(name, null, attrs);
                }
            }

            if (DEBUG) System.out.println("Created view is: " + view);
            return view;

        } catch (InflateException e) {
        //……
        }
    }
{% endhighlight %}

createViewFromTag()方法里面调用了Factory的onCreateView()或者LayoutInflater的onCreateView()方法或者createView()方法。事实上，createView()方法才是关键，因为它是正真创建一个View的方法。且看它的实现：

{% highlight java linenos %}
    public final View createView(String name, String prefix, AttributeSet attrs)
            throws ClassNotFoundException, InflateException {
        Constructor constructor = sConstructorMap.get(name);
        Class clazz = null;

        try {
            if (constructor == null) {
                // Class not found in the cache, see if it's real, and try to add it
                clazz = mContext.getClassLoader().loadClass(
                        prefix != null ? (prefix + name) : name);
                
                if (mFilter != null && clazz != null) {
                    boolean allowed = mFilter.onLoadClass(clazz);
                    if (!allowed) {
                        failNotAllowed(name, prefix, attrs);
                    }
                }
                constructor = clazz.getConstructor(mConstructorSignature);
                sConstructorMap.put(name, constructor);
            } else {
                // If we have a filter, apply it to cached constructor
                if (mFilter != null) {
                    // Have we seen this name before?
                    Boolean allowedState = mFilterMap.get(name);
                    if (allowedState == null) {
                        // New class -- remember whether it is allowed
                        clazz = mContext.getClassLoader().loadClass(
                                prefix != null ? (prefix + name) : name);
                        
                        boolean allowed = clazz != null && mFilter.onLoadClass(clazz);
                        mFilterMap.put(name, allowed);
                        if (!allowed) {
                            failNotAllowed(name, prefix, attrs);
                        }
                    } else if (allowedState.equals(Boolean.FALSE)) {
                        failNotAllowed(name, prefix, attrs);
                    }
                }
            }

            Object[] args = mConstructorArgs;
            args[1] = attrs;
            return (View) constructor.newInstance(args);

        } catch (NoSuchMethodException e) {
            //……
        }
    }
{% endhighlight %}

createView()方法会根据name参数，生成或者从系统内存中获取一个构造器：

Constructor constructor = sConstructorMap.get(name); 

或者constructor = clazz.getConstructor(mConstructorSignature);

然后通过构造器生成一个View对象，并且返回来：

{% highlight java linenos %}
    return (View) constructor.newInstance(args);
{% endhighlight %}

构造器调用newInstance(args)方法，并传入args参数，这时候就会调用对应的View类的构造函数。例如，当createView()方法的参数name是LinearLayout，则就会调用LinearLayout的构造函数。并且args[1] = attrs，传入了相关的属性集合。我们可以看看LinearLayout的带有attrs参数的构造函数：

{% highlight java linenos %}
    public LinearLayout(Context context, AttributeSet attrs) {
        super(context, attrs);

        TypedArray a = 
            context.obtainStyledAttributes(attrs, com.android.internal.R.styleable.LinearLayout);

        int index = a.getInt(com.android.internal.R.styleable.LinearLayout_orientation, -1);
        if (index >= 0) {
            setOrientation(index);
        }

        index = a.getInt(com.android.internal.R.styleable.LinearLayout_gravity, -1);
        if (index >= 0) {
            setGravity(index);
        }

        boolean baselineAligned = a.getBoolean(R.styleable.LinearLayout_baselineAligned, true);
        if (!baselineAligned) {
            setBaselineAligned(baselineAligned);
        }
        //……
        a.recycle();
    }
{% endhighlight %}

LinearLayout的构造函数首先会调用父类的构造函数super(context, attrs)，传入了相同的参数，然后再对LinearLayout特有的属性进行设置，如：

设置子控件排版方向的setOrientation(index);

设置子控件的对齐位置的setGravity(index);

设置基线对齐方式的setBaselineAligned(baselineAligned)。

我们要寻找的是设置一个View的高度和宽度，而LinearLayout是继承于View的，故高度和宽度的属性应该是在View的构造函数当中。那么不如一步步的跟踪LinearLayout的父类的构造函数，以及父类的的父类的构造函数……最终找到了View的构造函数：

{% highlight java linenos %}
    public View(Context context, AttributeSet attrs, int defStyle) {
        this(context);

        TypedArray a = context.obtainStyledAttributes(attrs, com.android.internal.R.styleable.View,
                defStyle, 0);

        Drawable background = null;

        int leftPadding = -1;
        int topPadding = -1;
        int rightPadding = -1;
        int bottomPadding = -1;

        int padding = -1;

        int viewFlagValues = 0;
        int viewFlagMasks = 0;

        boolean setScrollContainer = false;

        int x = 0;
        int y = 0;

        int scrollbarStyle = SCROLLBARS_INSIDE_OVERLAY;

        final int N = a.getIndexCount();
        for (int i = 0; i < N; i++) {
            int attr = a.getIndex(i);
            switch (attr) {
                case com.android.internal.R.styleable.View_background:
                    background = a.getDrawable(attr);
                    break;
                case com.android.internal.R.styleable.View_padding:
                    padding = a.getDimensionPixelSize(attr, -1);
                    break;
                 case com.android.internal.R.styleable.View_paddingLeft:
                    leftPadding = a.getDimensionPixelSize(attr, -1);
                    break;
        //……
{% endhighlight %}

在View的构造函数中，首先通过context.obtainStyledAttributes()方法返回一个TypedArray的对象数组，这个数组记录了相应的属性的值。然后使用for循环遍历这个数组，把这个数组里面的值取出来，并使用switch-case语句进行分派处理。例如处理background、padding等属性。

事实上在这个switch-case语句的分派处理中，并没有对View的高度和宽度的属性进行处理。这是为什么呢？找了半天了啊！还是没有找到这个东东，这是一个严峻的问题呢！

那我们好好回想一下怎样从xml文件中加载一个View呢？也就是那个rInflate(parser, temp, attrs)方法。

rInflate()方法的过程如下：

> * parser.getName()；
* createViewFromTag(name, attrs)；
* generateLayoutParams(attrs)；
* rInflate(parser, view, attrs)；
* addView(view, params)。

可以看到，调用了createViewFromTag(name, attrs)方法创建了一个View之后，还生成相应的LayoutParams参数，而在addView(view, params)的时候就把参数传进去。

那么我们好好看看LayoutParams这个类是怎样的。这个类其实就是ViewGroup类里面的LayoutParams类。

{% highlight java linenos %}
    public static class LayoutParams {
        @Deprecated
        public static final int FILL_PARENT = -1;

        public static final int MATCH_PARENT = -1;

        public static final int WRAP_CONTENT = -2;

        //……
        public int width;
        //……
        public int height;

        /**
         * Used to animate layouts.
         */
        public LayoutAnimationController.AnimationParameters layoutAnimationParameters;

        //……

        public LayoutParams(Context c, AttributeSet attrs) {
            TypedArray a = c.obtainStyledAttributes(attrs, R.styleable.ViewGroup_Layout);
            setBaseAttributes(a,
                    R.styleable.ViewGroup_Layout_layout_width,
                    R.styleable.ViewGroup_Layout_layout_height);
            a.recycle();
        }
        //……
        public LayoutParams(int width, int height) {
            this.width = width;
            this.height = height;
        }

        public LayoutParams(LayoutParams source) {
            this.width = source.width;
            this.height = source.height;
        }

        //……
        protected void setBaseAttributes(TypedArray a, int widthAttr, int heightAttr) {
            width = a.getLayoutDimension(widthAttr, "layout_width");
            height = a.getLayoutDimension(heightAttr, "layout_height");
        }

        //……
    }
{% endhighlight %}

嗯嗯！似乎可以看到了想看的的了，在LayoutParams类中就是定义了width和height两个属性，这两个属性就是用来标志View的宽度和高度的。无论是LinearLayout、RelativeLayout、ImageView、EditText都是由这两个属性来标志它们的宽度和高度的。

在rInflate()方法中调用createViewFromTag()方法创建了一个View之后，就调用viewGroup类的generateLayoutParams(attrs)方法来生成一个LayoutParams参数，这个方法的实现是这样的：

{% highlight java linenos %}
    public LayoutParams generateLayoutParams(AttributeSet attrs) {
        return new LayoutParams(getContext(), attrs);
    }
{% endhighlight %}

generateLayoutParams()方法调用LayoutParams构造函数生成一个LayoutParams对象；而LayoutParams构造函数构造函数会调用setBaseAttributes()方法来设置LayoutParams的width和height属性。

在每一个View类中有一个私有的成员变量：

{% highlight java linenos %}
    /**
     * The layout parameters associated with this view and used by the parent
     * {@link android.view.ViewGroup} to determine how this view should be
     * laid out.
     * {@hide}
     */
    protected ViewGroup.LayoutParams mLayoutParams;
{% endhighlight %}

故View的派生类，如LinearLayout、ImageView等都存在这样的变量。在rInflate()方法中，通过addView(view, params)方法来设置这个View的mLayoutParams成员的值。而addView()方法的设置mLayoutParams参数主要调用过程是：

> * addView(view, params);
* addView(child, -1, params);
* addViewInner(child, index, params, false);
* child.setLayoutParams(params);

最后会通过View类的setLayoutParams(params)方法来设置这个mLayoutParams成员的值。其中child是View类的对象。

上面创建一个View的过程，都只是初始化这个View的各个属性而已，还没有真正绘制这个View。那么在绘制这个View之前对这个mLayoutParams变量的width和height属性进行重新设置就可以达到了重新调整这个View的size的目的了！不是吗？

绘制这个View的方法是在draw()方法中，而draw()方法会调用onDraw()方法。那么我们可以写一个类继承于View，并且重写draw()方法或者onDraw()方法，在重写的方法中要调用父类的对应方法，并在父类方法的前或后加入一些打印信息，例如：

{% highlight java linenos %}
    @Override
    public void draw(Canvas canvas) {
        Log.i(TAG, "->draw()");
        super.draw(canvas);
    }
{% endhighlight %}

然后，再重写一些有可能是draw()方法之前调用的一些方法，例如：
{% highlight java linenos %}
    @Override
    public void setLayoutParams(android.view.ViewGroup.LayoutParams params) {
        Log.i(TAG, "->setLayoutParams()");
        super.setLayoutParams(params);
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        Log.i(TAG, "->onMeasure()");
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    }

    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        Log.i(TAG, "->onLayout()");
        super.onLayout(changed, l, t, r, b);
    }
{% endhighlight %}

因为View中的measure()方法和layout()方法是final方法，故不能重载，只能重载被measure()方法和layout()方法调用的onMeasure()方法和onLayout()方法。

然后把这个新定义的类运用在xml文件中。然后运行程序，就可以看到下面的输出信息：
![img](/assets/2012-12-28-android-measure.png)

那么，从上面的输出信息可以看出，setLayoutParams()方法先被调用，然后是onMeasrue()方法，接着是onLayout()方法，组后才到draw()方法。

我们再研究一下这个setLayoutParams()方法是在什么时候被调用。可以在重载的setLayoutParams()方法中加入如下的出错语句：

{% highlight java linenos %}
    @Override
	public void setLayoutParams(android.view.ViewGroup.LayoutParams params) {
		((String) null).toString();
		Log.i(TAG, "->setLayoutParams()");
		super.setLayoutParams(params);
	}
{% endhighlight %}

运行程序的时候，就会得到如下的出错信息：
![img](/assets/2012-12-28-android-measure-2.png)

从上面的输出信息可以看出来，setLayoutParams()，是在加载xml文件的时候在addView()的时候进行设置的。

然后再看看onMeasure()方法是什么时候被调用的。同理在onMeasure()方法中加入出错语句((String) null).toString()，运行程序得到如下信息：
![img](/assets/2012-12-28-android-measure-3.png)

从上面的输出信息可以看出来，ScaleImageView的父View先调用View的measure()方法，然后再调用父View的onMeasure()方法，接着在onMeasure()方法中ScaleImageView就会调用View的measure()方法和ScaleImageView的onMeasure()方法。但是，这也并不能清楚地了解到这个onMeasure()方法是怎么无端端地被调用呢！哈！

那么就研究研究setLayoutParams()是怎样的一个过程呢？

在View中setLayoutParams()的实现是这样的：

{% highlight java linenos %}
    public void setLayoutParams(ViewGroup.LayoutParams params) {
        if (params == null) {
            throw new NullPointerException("params == null");
        }
        mLayoutParams = params;
        requestLayout();
    }
{% endhighlight %}

其中requestLayout()方法是最关键的，调用了requestLayout()，就是要请求重新来绘画View的layout，且看这个方法的实现：

{% highlight java linenos %}
    public void requestLayout() {
        if (ViewDebug.TRACE_HIERARCHY) {
            ViewDebug.trace(this, ViewDebug.HierarchyTraceType.REQUEST_LAYOUT);
        }

        mPrivateFlags |= FORCE_LAYOUT;

        if (mParent != null && !mParent.isLayoutRequested()) {
            mParent.requestLayout();
        }
    }
{% endhighlight %}

这个方法把这个View中的mPrivateFlags变量添加了FORCE_LAYOUT的特性，然后再调用parent的requestLayout()方法。想到这里，大家是不是很奇怪，一个View的parent不也是一个View吗？如果也是View，那么还不是会调用这个requestLayout()方法？这是一种可能，但是如果是这样的话，那么只是给每个View的mPrivateFlags变量设置一些特性而已，并没有其他实质性的引发重绘layout的操作啊！所以这不科学啊！那么View派生的类呢？例如最外层的layout，如LinearLayout等，重写了requestLayout()方法，是不是这样呢？

事不宜迟，那就看看吧！看过才知道，原来LinearLayout并没有重写requestLayout()方法呢！对PhoneWindow有了解的都知道（如果不了解，可以查看前面的内容），LinearLayout是最外层的layout，FrameLayout次之，然后才是我们通过xml文件定义的View。现在连最外层的LinearLayout都没有重写这个requestLayout()方法，那么哪里去做这个真正的重绘layout的请求呢？

难题还真是多啊……不过不管怎样，难题肯定可以解决的！那么接下来就好好理解android系统的事件分派机制了。

可以在任意地方使程序报错，然后输出打印信息，例如在一个构造函数中报错：

{% highlight java linenos %}
    public ScaleImageView(Context context, AttributeSet attrs) {
        super(context, attrs);
        ((String) null).toString();
        Log.i(TAG, "->ScaleImageView()-after super()");
    }
{% endhighlight %}

之后输出的打印信息为：
![img](/assets/2012-12-28-android-measure-4.png)

在错误信息中，最后一行的阴影信息就是运行的虚拟机dalvik输出的信息，然后这样顺着这些输出信息从下往上的顺序层层地调用。如果你想好好的了解这个过程是怎样的，你可以根据这些输出信息去寻找对应的class文件或者java文件，然后打开文件找到相应的行，然后一步步地跟踪这个调用的过程。

例如android.app.ActivityThread.main(ActivityThread.java:4627)，这就需要找到ActivityThread这个类，或者找到这个类的源文件，源文件在源代码目录下去找，也就是在sources目录去找“android/app/”目录，然后再找ActivityThread.java这个文件。然后打开这个类，找到4627行，这一行就是出错的那一行。找到这一行之后，还可以继续跟踪下去，找到最终出错的那一行代码。

事实上，这里面比较重要的一个类就是ViewRoot了，这个类是干什么的呢？它的父类是Handler，实现的接口类是ViewParent，详细的了解可以看下面的连接：

> * [android的窗口机制分析------ViewRoot类](http://blog.csdn.net/windskier/article/details/6957901)
* [Android对Handler和ViewRoot的理解](http://hi.baidu.com/gaogaf/item/3f5008f06b742910d6ff8c49)
* [Android 窗口管理](http://www.blogjava.net/lihao336/archive/2010/11/24/338962.html)
* [android中View, Window, Activity, WindowManager，ViewRoot几者之间的关系](http://chriszeng87.iteye.com/blog/1096633)

事实上，当启动一个Activity时，会创建一个PhoneWindow类的Window对象，这个对象是Activity和整个View系统交互的接口。另外Activity还有一个WindowManager对象，因为WindowManager是一个接口，需要一个实现类来生成这样的一个对象，那就是WindowManagerImpl类，这个类管理着所在应用进程的窗口。另外WindowManagerImpl，创建一个ViewRoot来管理该窗口的根View。并通过ViewRoot.setView方法把该View传给ViewRoot。ViewRoot用于管理窗口的根View，并和global window manger进行交互。

所以ViewRoot是在整个控件树的最顶端，是一个逻辑的树顶。ViewRoot实现了ViewParent的各个方法，包括了requestLayout()方法，那我们来看看这个方法的实现：

{% highlight java linenos %}
    public void requestLayout() {
        checkThread();
        mLayoutRequested = true;
        scheduleTraversals();
    }
{% endhighlight %}

而requestLayout()方法调用了ViewRoot 的scheduleTraversals()方法，那么就来看看scheduleTraversals()方法的实现：

{% highlight java linenos %}
    public void scheduleTraversals() {
        if (!mTraversalScheduled) {
            mTraversalScheduled = true;
            sendEmptyMessage(DO_TRAVERSAL);
        }
    }
{% endhighlight %}

然后scheduleTraversals()方法会调用sendEmptyMessage(DO_TRAVERSAL)发送DO_TRAVERSAL消息给ViewRoot的消息队列那里去。再申明一下，ViewRoot的父类是Handler呢！

到这里，似乎很明了了。

View中setLayoutParams()会调用View的requestLayout()方法；

View的requestLayout()方法会调用parent的requestLayout()方法；

如果parent是一个View，那么会继续调用parent的requestLayout()方法；

直到获取到最顶端的parent，这时候，这个parent不是一个View，而是一个ViewRoot，那么调用parent的requestLayout()方法，就是调用ViewRoot的requestLayout()方法；

继而调用ViewRoot的scheduleTraversals()方法；

继而ViewRoot的sendEmptyMessage(DO_TRAVERSAL)方法，发送DO_TRAVERSAL消息给ViewRoot的消息队列。

那么接下来的就是系统的Looper来对这个消息队列进行处理了。Looper按顺序从消息队列里面取出消息，然后交由ViewRoot来进行消息的分派处理，也就是重载Handler的handleMessage()方法：

{% highlight java linenos %}
    @Override
    public void handleMessage(Message msg) {
        switch (msg.what) {
        case View.AttachInfo.INVALIDATE_MSG:
            ((View) msg.obj).invalidate();
            break;
        case View.AttachInfo.INVALIDATE_RECT_MSG:
            final View.AttachInfo.InvalidateInfo info = (View.AttachInfo.InvalidateInfo) msg.obj;
            info.target.invalidate(info.left, info.top, info.right, info.bottom);
            info.release();
            break;
        case DO_TRAVERSAL:
            if (mProfile) {
                Debug.startMethodTracing("ViewRoot");
            }

            performTraversals();

            if (mProfile) {
                Debug.stopMethodTracing();
                mProfile = false;
            }
        //...
{% endhighlight %}

在handleMessage()方法中，使用switch-case语句来进行分派处理，其中就有一项是针对DO_TRAVERSAL消息进行处理的，处理这个消息时，调用了performTraversals()方法，那么再看看这个方法的实现：
{% highlight java linenos %}
    private void performTraversals() {
        // cache mView since it is used so much below...
        final View host = mView;
    //……
    host.measure(childWidthMeasureSpec, childHeightMeasureSpec);
    //……
{% endhighlight %}
    
在performTraversals()方法中，获取mView的引用，mView就是在ViewRoot.setView()方法把Activity的根View传递进来的那个View。这个根View调用了measure()方法来自己进行size的测量。因为根View是一个GroupView的派生类的对象，所以measure()方法又会调用GroupView的onMeasure()方法，然后这个onMeasure()方法又会调用各个子View的measure()方法，如此类推，直到调用最末端的View的onMeasure()方法。

这就是onMeasure()方法的从头到尾的一个调用过程。明白了吗，哈？

同理可以研究这个onLayout()方法。类似地，在performTraversals()方法中，同样调用了根View的layout()方法，这个方法，也是这样调用onLayout()方法，然后调用各个子View的layout()方法，直到最末端的View。

而draw()函数竟然也是在performTraversals()方法里面被调用。这跟我一开始的想法是相符的，并且这几个方法被调用的顺序是：

> * measure();
* layout();
* draw();

事实上，在View的layout()方法中会调用setFrame()方法，这个方法又会调用invalidate()方法，这个方法就是请求进行重绘的一个方法。而invalidate()方法最后又会循环往parent那里去进行重绘请求，调用的方法是nvalidateChild()，最终到达ViewRoot。在ViewRoot中调用的重绘请求的方法是也是invalidateChild()，这个方法会调用ViewRoot的scheduleTraversals()方法，发送DO_TRAVERSAL消息。这样子最终就会调用到performTraversals()方法来对View进行重绘了。