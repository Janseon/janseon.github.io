---
date: 2013-01-12 20:30:00+00:00
layout: post
title:  "布局兼容策略"
categories: Android Layout
tags: Android 布局策略
excerpt: 我们需要考虑的因素有：手机设备和平板设备的一些兼容性。
---

我们需要考虑的因素有：手机设备和平板设备的一些兼容性。一般平板设备的分辨率比手机设备的分辨率大很多，并且密度density也不一样。在这里我们应该要好好的区分各个名词变量和单位所代表的意义。相关的名词变量有：屏幕尺寸、像素、分辨率、像素密度，相关的单位：px、pt、dip(dp)、sp等。详细了解这些内容，可以看看以下的相关文章：

* [Android上常见度量单位【xdpi、hdpi、mdpi、ldpi】解读](http://wenku.baidu.com/view/384cd7a40029bd64783e2ca1.html)
* [android 单位详解](http://wenku.baidu.com/view/1c1b4a0a16fc700abb68fc2f.html)

上面的一些文章，既介绍了android系统一些关于布局的名词和单位，也介绍了系统提供的一些解决不同分辨率和像素密度的适配方案。这个适配方案，主要是根据不同的设备分辨率或者像素密度来编写不同的布局文件，然后放到指定分辨率或者像素密度的文件夹下面，应用程序会根据自己所在设备的分辨率和密度来加载相应目录下的布局文件。这里就不详述了。

下面，我想说说本人的解决布局问题策略。这个策略是兼容所有的设备分辨率的，包括横屏，竖屏等。下面就详细道来。

首先说说，这个策略的最基本的原理，那就是根据屏宽和屏高来进行比例划分。当然，这里主要还是根据屏宽来比例划分的。首先说说，屏宽相关的内容。这里提出一个比例单位，叫做宽比，也就是某一宽度与屏宽的比例，数值在0-100之间（也可以自定义一个范围）。某一宽度与屏宽都是以像素点的个数来度量。例如在480*800分辨率的屏幕当中，屏宽=480，如果某一宽度为120，则这个宽度的宽比=120/480*100=25。如果，我们知道某一宽度的宽比为25，然后就可以相应的计算出该宽度=25/100*480=120。

那么相应的布局就可以根据这个0-100的数值来定义了。而对于fill_parent和wrap_content，照旧使用，我们可以设定一个值，分别是-1，-2。这个值是有根据的，是系统默认的。我们可以查看ViewGroup类的内部类LayoutParams：

{% highlight java linenos %}
    public static class LayoutParams {
        /**
         * Special value for the height or width requested by a View.
         * FILL_PARENT means that the view wants to be as big as its parent,
         * minus the parent's padding, if any. This value is deprecated
         * starting in API Level 8 and replaced by {@link #MATCH_PARENT}.
         */
        @SuppressWarnings({"UnusedDeclaration"})
        @Deprecated
        public static final int FILL_PARENT = -1;

        /**
         * Special value for the height or width requested by a View.
         * MATCH_PARENT means that the view wants to be as big as its parent,
         * minus the parent's padding, if any. Introduced in API Level 8.
         */
        public static final int MATCH_PARENT = -1;

        /**
         * Special value for the height or width requested by a View.
         * WRAP_CONTENT means that the view wants to be just large enough to fit
         * its own internal content, taking its own padding into account.
         */
        public static final int WRAP_CONTENT = -2;
        //……
{% endhighlight %}

那么一个View怎样去读取这个宽比，以及怎样根据这个宽比来测量出真正的高度呢？另外fill_parent和wrap_content怎样与-1和-2对应起来呢？要想回答这个问题，这就需要我们去了解系统是怎样去读取和测量这个高度的。

这可看看前面的章节“View的测量和重绘”。

假设我们都知道了View是怎样测量和重绘的，那么就可以根据这些测量和重绘过程来进行一些相关方法的重载，使得进行不同的处理。

第一种方法：重载setLayoutParams()方法。

我们都知道，View的size相关的属性width和height是在View的mLayoutParams变量里面记录着的。绘制这个View的size的时候，是根据这个mLayoutParams变量来进行绘制的。那么在绘制之前，调整好这个View的高度就可以了。而setLayoutParams()方法是在绘制之前调用的函数，所以可以重载这个函数来对View的size进行调整，如下代码：

{% highlight java linenos %}
    @Override
    public void setLayoutParams(ViewGroup.LayoutParams params) {
        viewWidth = params.width;
        viewHeight = params.height;
        int screenWidth = ScreenUtil.getScreenWidth(this.getContext());
        if (viewWidth > 100) {
            viewWidth = 100;
        }
        if (viewWidth > 0) {
            viewWidth = viewWidth / 100 * screenWidth;
        }
        if (viewHeight > 0) {
            viewHeight = viewHeight / 100 * screenWidth;
        }
        params.width = viewWidth;
        params.height = viewHeight;
        super.setLayoutParams(params);
    }
{% endhighlight %}

这样就可以实现了对View的调整。上面的策略是，设置View的宽度与屏幕宽度的比例是params.width/100，而高度与屏幕宽度的比例是params.height/100，其中前者的比例值不能大于1，而后者的比例值是可以大于1的。

同理，可以在onMeasure()方法中，进行size的调整：

{% highlight java linenos %}
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        ViewGroup.LayoutParams params = (ViewGroup.LayoutParams) this
                .getLayoutParams();
        if (params != null) {
            viewWidth = params.width;
            viewHeight = params.height;
            //……
        }
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    }
{% endhighlight %}

当然，也可以在重载的onLayout()方法中进行调整，甚至可以在invalidate()中去调整，但是，要值得注意的是，第一次去调整和第一次以后去调整可能会有冲突，或者有重复的处理。所以可以在重载函数中加入如下语句，这样就比较妥当：

{% highlight java linenos %}
    @Override
    public void setLayoutParams(ViewGroup.LayoutParams params) {
        if (!isFirstToSetLayoutParams) {
            super.setLayoutParams(params);
        }
        isFirstToSetLayoutParams = false;
        
        viewWidth = params.width;
        viewHeight = params.height;
        //……
{% endhighlight %}

另外，还要注意是在xml文件中layout_width和layout_height的单位，如果用px作为单位，那么在重载的方法中进行处理就简单一点，因为不需要其他的单位变换了。如果用其他的单位，那么就需要做单位转化的处理。那么单位怎么进行处理呢？其实系统里面有这样的一个方法来进行单位的变换处理，当然自己也可以参考那个方法来实现自己方法。

从GroupView的LayoutParams类中找到setBaseAttributes()方法：

{% highlight java linenos %}
    protected void setBaseAttributes(TypedArray a, int widthAttr, int heightAttr) {
        width = a.getLayoutDimension(widthAttr, "layout_width");
        height = a.getLayoutDimension(heightAttr, "layout_height");
    }
{% endhighlight %}

可以看到TypedArray 的getLayoutDimension()方法，这个方法里面就有可能包括了关于单位转换的方法了，跳进去看看，然后找到TypedValued的complexToDimensionPixelSize()方法，然后沿着这个方法继续跳进去，找到applyDimension()方法，这个就是传说中单位转化方法了，看看它的实现：

{% highlight java linenos %}
    public static float applyDimension(int unit, float value, DisplayMetrics metrics) {
        switch (unit) {
        case COMPLEX_UNIT_PX:
            return value;
        case COMPLEX_UNIT_DIP:
            return value * metrics.density;
        case COMPLEX_UNIT_SP:
            return value * metrics.scaledDensity;
        case COMPLEX_UNIT_PT:
            return value * metrics.xdpi * (1.0f/72);
        case COMPLEX_UNIT_IN:
            return value * metrics.xdpi;
        case COMPLEX_UNIT_MM:
            return value * metrics.xdpi * (1.0f/25.4f);
        }
        return 0;
    }
{% endhighlight %}

各个单位的值转化为px单位的值的计算公式就在这个函数里面了。其中px单位是系统真正能够辨别的标准单位。

所以感觉最就用px单位来进行调整，如果用其他单位的话，那么就记得转换单位哦，亲！

刚才已经使用了一种策略来进行布局。这种“策略一”主要是：以屏幕的宽度，也就是以屏宽为总的度量长度，View的宽和高的最终值确定于与屏宽的比例。这个比例值可以在xml文件的layout_width和layout_height的属性中设定，单位建议用px。

再提出另外的策略。“策略二”主要是：以屏幕的宽度，也就是以屏宽为总的度量长度，View的宽的最终值确定于与屏宽的比例，而高度和宽度的比例与背景图或者前景图的宽高比是相等的。实现起来是这样的：

{% highlight java linenos %}  
    @Override
    public void setLayoutParams(ViewGroup.LayoutParams params) {
        viewWidth = params.width;
        viewHeight = params.height;
        int screenWidth = ScreenUtil.getScreenWidth(this.getContext());
        if (viewWidth > 100) {
            viewWidth = 100;
        }
        if (viewWidth > 0) {
            viewWidth = viewWidth / 100 * screenWidth;
        }
        params.width = viewWidth;
            
        final Drawable bgDrawable = this.getBackground();
        Drawable curDrawable = bgDrawable.getCurrent();
        if (curDrawable instanceof BitmapDrawable) {
            Bitmap bitmap = ((BitmapDrawable) curDrawable).getBitmap();
            float scale = viewWidth / (float) bitmap.getWidth();
            viewHeight = (int) (bitmap.getHeight() * scale);
            params.height = viewHeight;
        }
        super.setLayoutParams(params);
    }
{% endhighlight %}

通过获取背景图的的Bitmap的宽高比来设定View的尺寸。如果这个View是ImageView，那么可以添加以下的代码，进行前景图的适应尺寸，添加的代码如下：
    
{% highlight java linenos %}
    @Override
    public void setLayoutParams(android.view.ViewGroup.LayoutParams params) {
        viewWidth = params.width;
        int screenWidth = ScreenUtil.getScreenWidth(this.getContext());
        if (viewWidth > 100) {
            viewWidth = 100;
        }
        if (viewWidth > 0) {
            viewWidth = viewWidth / 100 * screenWidth;
        }
        params.width = viewWidth;
        final Drawable curDrawable = this.getDrawable().getCurrent();
        if (curDrawable instanceof BitmapDrawable) {
            Bitmap bitmap = ((BitmapDrawable) curDrawable).getBitmap();
            float scale = bitmap.getHeight() / (float) bitmap.getWidth();
            viewHeight = (int) (viewWidth * scale);
            params.height = viewHeight;
        }
        super.setLayoutParams(params);
    }
{% endhighlight %}

而“策略三”主要是：某个View以它的parent的宽度，也就是以parentWidth为度量长度，View的width和height的最终值确定于与parentWidth的比例。

因为在LayoutInflater类的rInflate()方法中，创建一个View的过程是这样的：

{% highlight java linenos %}
    final View view = createViewFromTag(name, attrs);
    final ViewGroup viewGroup = (ViewGroup) parent;
    final ViewGroup.LayoutParams params = viewGroup.generateLayoutParams(attrs);
    rInflate(parser, view, attrs);
    viewGroup.addView(view, params);
{% endhighlight %}

在addView()之前，就调用了rInflate()自己进行递归，也就是说是在创建一个View的时候，在addView()之前就先递归生成这个View的所有子View。故addView()的顺序是从最末端的子View开始往最顶端的根View进行调用的，同理addView()方法里面调用了setLayoutParams(params)方法，故setLayoutParams(params)方法的调用顺序是和addView()方法是一致的。

也就是说当某个View调用setLayoutParams(params)方法时，并没有通过它的parent进行addView()，故这时候是不存在parent的。

所以在重写的setLayoutParams(params)方法里面实现策略三是不合理的。

而draw()方法是与addView()和setLayoutParams(params)方法是相反的顺序的。draw()方法首先是在最顶端的ViewRoot那里先被调用的，其调用过程是这样的：

> * 首先调用ViewRoot的draw()方法;
* ViewRoot的draw()方法里面会调用根View的draw()方法；
* 根View的draw()方法会调用根View的dispatchDraw()方法；
* 因为根View是ViewGroup类或者派生类的实例，所以dispatchDraw()方法是ViewGroup类的dispatchDraw()方法；
* ViewGroup类的dispatchDraw()方法会循环调用它的子View的drawChild()方法;
* 子View的drawChild()方法又会调用该子View的dispatchDraw()或者draw()方法；
* 然后如此类推，直到最末端的子View。

所以，如果是依据parent来进行View尺寸的调整时，可以在重写draw()方法中进行。例如：

{% highlight java linenos %}
    @Override
    public void draw(Canvas canvas) {
        ViewGroup.LayoutParams params = this.getLayoutParams();
        if (!isFirstToSetLayoutParams || params == null) {
            super.draw(canvas);
        }
        isFirstToSetLayoutParams = false;
        viewWidth = params.width;
        viewHeight = params.height;
        View parent = (View) this.getParent();
        int parentWidth = 0;
        if (parent == null) {
            parentWidth = ScreenUtil.getScreenWidth(this.getContext());
        } else {
            parentWidth = parent.getWidth();
        }
        if (viewWidth > 100) {
            viewWidth = 100;
        }
        if (viewWidth > 0) {
            viewWidth = viewWidth / 100 * parentWidth;
        }
        if (viewHeight > 0) {
            viewHeight = viewHeight / 100 * parentWidth;
        }
        params.width = viewWidth;
        params.height = viewHeight;
        super.draw(canvas);
    }
{% endhighlight %}

那么在其他的重写的方法上可以吗？例如：onMeasure()方法中呢？onLayout()方法呢？

这个，本人没有测试过，但是onMeasure()方法、onLayout()方法与draw()方法是很类似的，大家可以用同样的方法来检验是否能达到我们的要求，在这里就不多说了。


另外，我们需要考虑的是padding和margin相关的值。先要知道padding参数是在View的构造函数中进行设置的：

{% highlight java linenos %}
    public View(Context context, AttributeSet attrs, int defStyle) {
        //……
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
            case com.android.internal.R.styleable.View_paddingTop:
                topPadding = a.getDimensionPixelSize(attr, -1);
                break;
            case com.android.internal.R.styleable.View_paddingRight:
                rightPadding = a.getDimensionPixelSize(attr, -1);
                break;
            case com.android.internal.R.styleable.View_paddingBottom:
                bottomPadding = a.getDimensionPixelSize(attr, -1);
                break;
        //……
{% endhighlight %}

我们可以在View的构造函数之后就重新设置padding的值：

{% highlight java linenos %}
    public SImageView(Context context, AttributeSet attrs) {
        super(context, attrs);
        int screenWidth = ScreenUtil.getScreenWidth(context);
        setPadding( // 重新设置各边的padding
                getPaddingLeft() * screenWidth / 100, // 重新设置paddingLeft
                getPaddingTop() * screenWidth / 100, // 重新设置paddingTop
                getPaddingRight() * screenWidth / 100,// 重新设置paddingRight
                getPaddingBottom() * screenWidth / 100// 重新设置paddingBottom
        );
    }
{% endhighlight %}

而getPaddingLeft()等方法返回的值是存在的，因为在View的构造函数中调用过setPadding()方法来进行设置了。故这是可行的。在xml文件中，建议的值是在0-100，单位是px。

而对于margin的值，因为View的margin值是属于ViewGroup类的MarginLayoutParams类里面的，所以最好是在setLayoutParams()的重载方法里面进行重设。在各个layout中，如LinearLayout和RelativeLayout等，里面都重新定义了一个内部类LayoutParams，这个类都继承了ViewGroup类的MarginLayoutParams类；故可以通过设置LayoutParams参数的margin值来调整这些layout的边缘大小。

事实上，在LayoutInflater类的rInflate()方法中，创建一个View的过程是这样的：

{% highlight java linenos %}
    final View view = createViewFromTag(name, attrs);
    final ViewGroup viewGroup = (ViewGroup) parent;
    final ViewGroup.LayoutParams params = viewGroup.generateLayoutParams(attrs);
    rInflate(parser, view, attrs);
    viewGroup.addView(view, params);
{% endhighlight %}

获取这个View在parent中的参数的方法是viewGroup.generateLayoutParams(attrs)；这个方法是先在ViewGroup类里面定义的，而其他的layout，如LinearLayout等都重写了这个方法，如在LinearLayout中重写的这个方法为：

{% highlight java linenos %}
    @Override
    public LayoutParams generateLayoutParams(AttributeSet attrs) {
        return new LinearLayout.LayoutParams(getContext(), attrs);
    }
{% endhighlight %}

返回了一个LinearLayout的LayoutParams的一个实例对象。根据java的多态性，如果通过parent是一个LinearLayout的实例，也就是viewGroup 是一个LinearLayout的实例，那么获取参数的方法viewGroup.generateLayoutParams(attrs)，获取到的参数就是LinearLayout的LayoutParams的一个实例。所以在创建一个View的过程中，会根据相应的类型调用相应的重写方法，进行相应的处理的；这就是编程当中多态的一些魅力。

而viewGroup.addView(view, params)方法会调用View的setLayoutParams()方法来进行参数的设置。所以如果要调整view的margin值，那么最好是在setLayoutParams()方法中，重新调整LayoutParams参数的margin值，所以可以在重写的setLayoutParams()方法添加以下的代码：

{% highlight java linenos %}
    @Override
    public void setLayoutParams(android.view.ViewGroup.LayoutParams params) {
        //……
        if (params instanceof ViewGroup.MarginLayoutParams) {
            ViewGroup.MarginLayoutParams marginParams = (ViewGroup.MarginLayoutParams) params;
            marginParams.setMargins( // 重新设置margin
                    marginParams.leftMargin * screenWidth / 100,
                    marginParams.topMargin * screenWidth / 100,
                    marginParams.rightMargin * screenWidth / 100,
                    marginParams.bottomMargin * screenWidth / 100/;
            );
        }
        super.setLayoutParams(params);
    }
{% endhighlight %}

这样子就重新设置了margin的值了。