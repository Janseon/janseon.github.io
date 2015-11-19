---
date: 2012-12-20 20:30:00+00:00
layout: post
title:  "多级列表"
categories: Android List
tags: Android List Adapter
excerpt: 所谓多级列表，首先是一个列表，如果这个列表的列表项又是一个列表，称为二级列表
---

1.认识
------
所谓多级列表，首先是一个列表，如果这个列表的列表项又是一个列表，称为二级列表，如果这个二级列表的子项又是一个列表，那么这个列表称为三级列表，如此类推……

多级列表的用途还是很广的，如一条微博或者一片文章有N条评论，一条评论又包含M条回复，如果要把微博、评论和回复都显示出来的话，那就需要多级的列表来显示，如下：

* 微博1
* * 评论1
* * * 回复1
* * * 回复2

……

* * 评论2
* * * 回复1
* * * 回复2

……

……

* 微博2
* * 评论1
* * * 回复1
* * * 回复2
……

* * 评论2
* * * 回复1
* * * 回复2

……

……


以上就是一个3级的列表，虽然3级列表很少用到，一般用到2级列表就足够了，但是随着设备屏幕的变大，用户的需求的增加，还是可能会用到的，例如我公司多益的火星的CC Android版的企业群，就是这样的一个多级列表，群上面有子群，子群又有子群，如此类推。
<p></p>

2.二级列表
------
那么怎样实现多级列表呢，根据多级列表的定义，可以试试在列表控件的列表项里面嵌套一个列表控件；就先拿二级列表来说明，如下：

* 微博1
* * 评论1
* * 评论2
* 微博2
* * 评论1
* * 评论2

详细的代码可以参考：


但是，这样嵌套列表控件，会引发很多莫名其妙的问题，例如可能会提示layout嵌套的层次太深，也可能会使得整个列表控件显示的内容不完整，也有可能使得第二级列表的子项上的控件有时候会扑捉不到焦点。这些都是很严重的问题，使得不能继续开发下去了。对于layout嵌套的层次太深的问题，可以适当的减少LinearLayout、RelativeLayout的使用和嵌套，使得布局文件更轻量级（**建议1**）；对于列表控件显示不完整，这就需要重新计算第二级的列表的各个子项和头、driver条的高度，然后加起来，作为第一级列表的子项的高度（建议2）；对于在第二级列表的子项上的控件扑捉不到焦点，这个到现在还没有解决这个问题，因为本人当时就摈弃了列表嵌套列表的的这种方法。

**建议1**：事实上本人是很提倡去减少LinearLayout、RelativeLayout的使用和嵌套的，因为系统加载LinearLayout、RelativeLayout等布局的消耗是不小的，嵌套的层次越多，启动的速度就越慢，所以，要想办法用尽可能少的layout布局去实现自己想要的布局；当然，有时候需要include来分割布局文件，使得代码的机构模块化，这就需要加多一些布局，这也是有好处的，因此，凡事都不能绝对，都需要折中考虑。

**建议2**：当ScrollView嵌套ListView，或者嵌套带有滚动条的容器控件时，同样需要重新计算嵌套在里面的控件的高度来使得控件的内容完整显示；个人觉得，控件嵌套控件之后，竟然需要通过开发者去写程序重新计算控件的高度，来告诉Android系统里面的控件的高度发生了变化，然后Android系统才显示正确的高度；这样做似乎有些不妥呢！！唉唉，不知这样做是Android系统太灵活了，还是太不人性化了，这算是Android系统的优点还是缺点呢？总之嘛，本人对这样做，有种蛋疼的感觉；随着自己的技术的提升，慢慢地发现，Android系统可能是不希望开发者这样用一些大消耗的控件互相嵌套（这里的大消耗的控件是指带有滚动条的容器控件如ScrollView、ListView等，而LinearLayout、RelativeLayout这些消耗没有ScrollView嵌套ListView大）；但也提供了重新计算的高度的方法来以备不时之需；事实上，很多这些嵌套都是多余的，因为基本上会有其他更好的方法来替代，甚至替代后会有更好的性能；例如ScrollView嵌套一个ListView，可以用一个ListView添加一个headerView来替代；ScrollView嵌套多个ListView，可以用二级列表替代，或者ListView使用LinearLayout替代等，这就需要具体情况来进行改变；那么ListView嵌套ListView可以怎样替代呢？请看下文……

记得一开始，我就使用了一个ListView，这个ListView的子项就是评论，每条评论的子项就是回复；这个ListView的item项，也就是一条评论，是一个LinearLayout，那么怎样在这条LinearLayout评论上增加回复的内容呢？没错，就是用addView()进行添加，有多少回复就在这个LinearLayout操作多少次addView()。

起初这个方法是可行的，而且还算流畅，但是随着数据的增多，评论和回复数据的增加，主要是回复的数据的增加，使得这个页面的负载变得很重；以至于后来，评论内容上的EditText控件突然获取不了焦点，更不要谈回复上的EditText控件了。

究其原因，个人觉得是因为在LinearLayout操作addView()时，彻底违反了Android系统的可重用机制；这个可重用机制是从ListView的实现原理里面得来的；ListView是一个容器控件，里面装着要显示的子项的内容，那么ListView到底有多少个子项呢？例如ListView要显示100条评论，那么ListView是有100个子项吗？答案是错的！那到底有多少的子项？那就要看子项到底有多高，例如5个子项加起来的高度刚好是ListView能显示的高度，那么这时候ListView的子项个数就比5多一个或几个，至于几个，需要Android系统会根据具体情况来控制，不过肯定没有100个，而你也可以根据ListView的getChildCount()方法，不断的获取他的子项数目并且输出来验证，如下图：
![img](/assets/2012-12-20-multilevel-list.png)

那么为什么会是这样的情况呢？原来ListView继承于AbsListView，而AbsListView的内部有一个RecycleBin，专门用来回收使用过的子项，而ListView的适配器重写的函数getView(final int position, View convertView, ViewGroup parent)的参数convertView就是从这个回收箱RecycleBin里面取出来的，一开始是RecycleBin是空的，所以convertView参数是空的，当RecycleBin不为空，且还存在可用的内容，并且View的类型是对应的，那么就会传给convertView，这时候getView的参数convertView就不为空，并且被LitView重用了；而ListView生成子项的过程是一系列的removeView()，removeAllViews()和setupChild()过程，使得ListView的子项个数和屏幕可显示的子项数是对应的；注意，这里是setupChild()，而不是addView()，因为ListView不支持addView()，只支持私有的setupChild()。具体的内容，可以自己好好研究ListView的实现细节，会大有脾益的。提示：从ListView的方法跟踪：

其中，filldown(int pos, int nextTop)函数是从pos位置到nextTop位置填充各子项；

{% highlight java linenos %}
    filldown()->
        makeAndAddView()->
            AbsListView.obtainView()->
                mAdapter.getView(); //适配器的getView()方法就在这里调用的！！
    setupChild();
{% endhighlight %}

因此，在ListView上使用getChild()，getChildCount()方法基本是没什么用的，一般不使用。

也可以参考这个文章来感悟一下这种重用机制的性能影响：
* [listView的可重用机制对性能的影响](http://www.cnblogs.com/xitang/archive/2011/09/15/2177318.html)

回到评论的LinearLayout那里去说，如果回复很多，LinearLayout就需要不断的addView来显示回复的内容，也就是说，如果有100条回复，那么就需要操作100次addView，并且生成100个子项，这样做毋庸置疑，肯定会使得评论的LinearLayout负载过重，性能变得低下。

那么还有没有好办法呢？上面提及的LinearLayout的不断addView之所以性能不好，是因为违反了再重用机制，那么能不能把再重用机制用到这个LinearLayout上面去呢？答案是肯定的，这样的思路也很接近本人的最佳方案了，只是这样做是比较复杂的一件事情！因为你要考虑这些再重用的View什么时候remove；而在ListView中，系统会自动判断哪些View已经不需要显示，而remove掉并且放进回收箱；但是在这里的LinearLayout就需要自己去判断哪些View已经不需要显示了，这个过程当然可以模仿ListView去实现，但是这个判断的过程是不简单的，所以这样会把原来的问题弄得更复杂了，这，得不偿失啊。

作为一个有心人，本人曾对ListView进行了深入的研究，最后得知ListView的父类AbsListView中RecycleBin的再回收机制是支持多种不同的View的类型的；View的参数中AbsListView.LayoutParams的类中有一个viewType，用来标记这个View的类型；

在RecycleBin中有一个mScrapViews变量：

{% highlight java linenos %}
    private ArrayList<View>[] mScrapViews;
{% endhighlight %}

mScrapViews变量是一个数组，View的类型有多少种，mScrapViews数组就有多大，View的类型viewType是一个int类型，对应mScrapViews数组的索引，如果View的类型是viewType，则在回收View的时候，会有这样的操作：

{% highlight java linenos %}
    mScrapViews[viewType].add(View);
{% endhighlight %}

详细可以看看addScrapView()的代码，如下：

{% highlight java linenos %}
    void addScrapView(View scrap) {
        AbsListView.LayoutParams lp = (AbsListView.LayoutParams) scrap.getLayoutParams();
        if (lp == null) {
            return;
        }

        // Don't put header or footer views or views that should be ignored
        // into the scrap heap
        int viewType = lp.viewType;
        if (!shouldRecycleViewType(viewType)) {
            if (viewType != ITEM_VIEW_TYPE_HEADER_OR_FOOTER) {
                removeDetachedView(scrap, false);
            }
            return;
        }

        if (mViewTypeCount == 1) {
            scrap.dispatchStartTemporaryDetach();
            mCurrentScrap.add(scrap);
        } else {
            scrap.dispatchStartTemporaryDetach();
            mScrapViews[viewType].add(scrap);
        }

        if (mRecyclerListener != null) {
            mRecyclerListener.onMovedToScrapHeap(scrap);
        }
    }
{% endhighlight %}

而从回收箱里面取出来viewType类型的view就会有类似于这样的操作：

{% highlight java linenos %}
    View = mScrapViews[viewType].remove(0);
{% endhighlight %}

详细可以看看getScrapView()的代码，如下：

{% highlight java linenos %}
    View getScrapView(int position) {
        ArrayList<View> scrapViews;
        if (mViewTypeCount == 1) {
            scrapViews = mCurrentScrap;
            int size = scrapViews.size();
            if (size > 0) {
                return scrapViews.remove(size - 1);
            } else {
                return null;
            }
        } else {
            int whichScrap = mAdapter.getItemViewType(position);
            if (whichScrap >= 0 && whichScrap < mScrapViews.length) {
                scrapViews = mScrapViews[whichScrap];
                int size = scrapViews.size();
                if (size > 0) {
                    return scrapViews.remove(size - 1);
                }
            }
        }
        return null;
    }
{% endhighlight %}

所以我们在写ListView的适配器类的时候就需要定义两种类型的View，如下：

{% highlight java linenos %}
    class ListAdapter extends BaseAdapter {
        //……
        final int VIEW_TYPE_GROUP = 0;
        final int VIEW_TYPE_CHILD = 1;
    //……
{% endhighlight %}

那么，什么时候设置View的类型呢？

通过研究ListView的实现代码，原来在ListView的setupChild()方法中有这样的表达式：

{% highlight java linenos %}
    p.viewType = mAdapter.getItemViewType(position);
{% endhighlight %}

那么从回收箱里面获取某个位置上的View是怎么确定该View的类型呢？查看一下getScrapView()的代码，原来还是mAdapter.getItemViewType(position)：

{% highlight java linenos %}
    int whichScrap = mAdapter.getItemViewType(position);
{% endhighlight %}

而getItemViewType是ListView适配器的一个方法，那么我们就可以重写ListView适配器ListAdapter的getItemViewType方法了，另外还要对应地重写getViewTypeCount()，这个方法说明总共多少种View。

如下：

{% highlight java linenos %}
    @Override
    public int getItemViewType(int position) {
        if (list.get(position).isGroup) {
            return VIEW_TYPE_GROUP;
        } else {
            return VIEW_TYPE_CHILD;
        }
    }
    @Override
    public int getViewTypeCount() {
        return typeCount;
    }
{% endhighlight %}

其中typeCount=2，表示有两种View，大家肯定会好奇，list.get(position).isGroup这个到底是什么？哈哈……先不要急，我慢慢道来！

先定义几个类：

{% highlight java linenos %}
    public class Reply { {
        public String content;
    }
    public class Comment { {
        public String content;
        public ArrayList<Reply> replyList;
    }
    public class Pos {
        public boolean isGroup = false;
        public Reply reply = null;
        public Comment comment = null;
    }
{% endhighlight %}

现在，我们可以看看ListAdater定义了哪几个变量：

{% highlight java linenos %}
    class ListAdapter extends BaseAdapter {
        private Context context;
        private LayoutInflater inflater;
        private ArrayList<Pos> list;

        int typeCount = 2;
        final int VIEW_TYPE_GROUP = 0;
        final int VIEW_TYPE_CHILD = 1;
    //……
{% endhighlight %}

而getView()，又是怎样实现的呢？且看：

{% highlight java linenos %}
    public View getView(final int position, View convertView, ViewGroup parent) {
        Pos pos = list.get(position);
        if (pos.isGroup) {
            convertView = getGroupView(pos, convertView, parent);
        } else {
            convertView = getChildView(pos, convertView, parent);
        }
        return convertView;
    }
{% endhighlight %}

其中getGroupView和getChildView是自己自定义的方法，主要是让代码结构清晰，getGroupView代表的是获取VIEW_TYPE_GROUP类型，也就是comment类型的View，而getChildView代表的是获取VIEW_TYPE_CHILD类型，也就是reply类型的View。

到现在估计大家都能够清楚明白list.get(position).isGroup是一个什么东东了吧！

现在给大家讲解一下实现的原理。事实上，通过ListAdapter实现的ListView只是一个平铺的ListView，无论是group类型的还是child类型的item都是ListView的子项，每个子项与ListAdapter的list的每一个元素一一对应，list中的元素是Pos对象，Pos对象记录了该子项是group类型还是child类型；当是group类型时，对应的View就显示comment数据，当是child类型时，对应的View就显示reply数据；这样就形成了和二级列表一样显示效果的页面。

至于getView那里的那个参数convertView，如果不为空，那到底对应的是类型呢？又是怎样对应到正确的类型呢？我们可以看看ListView中的makeAndAddView()方法，这个方法调用了AbsListView类的obtainView()方法，而这个方法调用了mAdapter.getView()方法；getView()方法就是在这里被调用的，我们研究一下getView()被调用的前后，如下：

{% highlight java linenos %}
    View obtainView(int position, boolean[] isScrap) {
        isScrap[0] = false;
        View scrapView;

        scrapView = mRecycler.getScrapView(position); //在getView()方法被调用之前

        View child;
        if (scrapView != null) {
            if (ViewDebug.TRACE_RECYCLER) {
                ViewDebug.trace(scrapView, ViewDebug.RecyclerTraceType.RECYCLE_FROM_SCRAP_HEAP,
                        position, -1);
            }

            child = mAdapter.getView(position, scrapView, this);

            if (ViewDebug.TRACE_RECYCLER) {
                ViewDebug.trace(child, ViewDebug.RecyclerTraceType.BIND_VIEW,
                        position, getChildCount());
            }

            if (child != scrapView) {
                mRecycler.addScrapView(scrapView);
                if (mCacheColorHint != 0) {
                    child.setDrawingCacheBackgroundColor(mCacheColorHint);
                }
                if (ViewDebug.TRACE_RECYCLER) {
                    ViewDebug.trace(scrapView, ViewDebug.RecyclerTraceType.MOVE_TO_SCRAP_HEAP,
                            position, -1);
                }
            } else {
                isScrap[0] = true;
                child.dispatchFinishTemporaryDetach();
            }
        } else {
            child = mAdapter.getView(position, null, this);
            if (mCacheColorHint != 0) {
                child.setDrawingCacheBackgroundColor(mCacheColorHint);
            }
            if (ViewDebug.TRACE_RECYCLER) {
                ViewDebug.trace(child, ViewDebug.RecyclerTraceType.NEW_VIEW,
                        position, getChildCount());
            }
        }

        return child;
    }
{% endhighlight %}
在getView()方法被调用之前，先被mRecycler.getScrapView(position)，而前面也说过，getScrapView()方法也会根据mAdapter.getItemViewType(position)方法获得对应position位置的View的类型，然后根据这个类型，从回收箱里面取出回收了的View供当前使用，如果取出来的为空，则getView()方法传入的参数convertView就为空。

“原来如此！”，是不是有这样的一种感叹呢？哈……详细的列子可以查看文档文件夹里面的src文件夹里面的CommentsActivity.java文件。

事实上根据这样的原理，我们可以自定义三级列表、四级列表，甚至更多级的的页面，只要设计一个对应级数的Pos类，以及重写适配器的getView，getItemViewType，getViewTypeCount方法就可以了。


到后来，本人突然要用到ExpandableListView这个容器控件，为了更好的使用这个控件，我深入去查看这个控件的源代码，发现ExpandableListView控件作为一个二级列表的一种实现和我之前的的想法具有异曲同工之妙，里面同样使用了类似于一个Pos的类，在ExpandableListView这个类是PositionMetadata；到这里我我不禁感叹自己的这神奇的想法，既简单又完美，哈，这不就是我追求的“简单美”吗？

另外，ExpandableListView实现起来复杂很多，需要更多的类更多的文件，其中适配器就需要两个了。只是ExpandableListView的复杂也是有原因的，因为它实现的功能比我实现的多很多。不过我实现的二级列表的功能已经够用了。并且扩展性也很好，可以扩展到三级甚至四级，然后还可以根据需要添加新功能，所以嘛，自己定义的东西就是简单、方便、自由、灵活啊……这也是Android开发的很大的魅力所在呢！


**总结**：实现一个功能，或者实现一个效果，是多种方法的，我们必须要多尝试和多思考，在思考的过程中，我们要遵循“简单美”这样的原则，想出既简单，要漂亮的方法来解决一些难题。有时候不一定要完全靠自己去想，你可以依靠各种途径去找灵感，可以通过网络、书籍各种资料，也可以跟别人交流，有时候别人说的一句话，都可能让你突发灵感；还可以通过研究别人的项目代码，或者SDK的源代码，因为这些是前人的心血，通过阅读别人的杰作，然后总结出自己的简单的东西出来，这样子，咱一定可以受益匪浅的。