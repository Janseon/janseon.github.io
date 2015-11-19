---
date: 2013-02-12 20:30:00+00:00
layout: post
title:  "缓存机制"
categories: Android Utils
tags: Android 学习
excerpt: 这里说的缓存一般可以分为普通数据的缓存，以及图片缓存
---

这里说的缓存一般可以分为普通数据的缓存，以及图片缓存。

普通数据的缓存，这个比较好办，主要是把下次需要显示的数据以文件的形式储存在android设备的本地磁盘中，然后下次使用的时候，从本地磁盘的相应文件读取并加载进来显示就可以了。不过，需要注意的是数据保存的格式要保持一致性。例如，你把数据以json格式储存，那么加载这些缓存数据的时候就需要以json格式来读取，如果以xml格式来储存的，就要以xml的格式来读取。现在比较常用的是json格式的数据。主要原因是服务器那边常用这些json数据格式返回给android客户端，所以嘛，客户端这边就顺手把这些数据储存起来就行啦，哈！

另外，有一样比麻烦的是，如果要对json数据文件添加一些数据，那么到底要在文件的哪里添加进去才是正确的呢？哈哈，这个嘛！就只能投机取巧，先把json数据加载进来生成相应的数据对象，然后在这个数据对象上进行添加，当添加完之后，再把这些数据对象变成json数据，然后储存起来。这样就需要我们去做一些工作，那就是把一些数据对象序列化成json的数据流。不过，序列化是程序员的必修课，而且也不难，估计这不是个重要问题。所以……就继续讲下去……

接下来就讲讲图片缓存，图片缓存也可以理解为Bitmap缓存。在Android中，图片以一般以Bitmap的格式存在于内存中。而传说中啊，Bitmap是一个超级大胖子呢……有多胖？胖到它会报出OOM（OUT OF MEMORY）的错误；不过这种情况都是因为内存里面存有大量的Bitmap才会报出这样的错误，并且这种错误事实上是Bitmap溢出的错误。

android系统中读取位图Bitmap时。分给虚拟机中图片的堆栈大小只有8M，这个8M的数据据说也不是很准确，有的说8M是最小的值，有的说堆栈的限制值必8M大很多，这可以参考以下链接：

* [android bitmap内存限制](http://phenom.iteye.com/blog/1679198)
* [android bitmap内存限制](http://www.apkbus.com/android-77791-1-1.html)

不管怎样，总之一个虚拟机对于内存是有限制的，一个应用基于一个虚拟机来运行，自然也就不能无限度的使用内存，故Bitmap这个大胖子也会有相应的限制。所以不管是如何加载图片，太多太大的Bitmap，虚拟机如果受不了了，肯定会报那个错误。超出图片内存预算，那么就会出现错误：java.lang.OutOfMemoryError: bitmap size exceeds VM budget。遇到这个问题时，就证明了没有及时回收资源。所以有必要去手动干预去释放掉一些Bitmap：

{% highlight java linenos %}
    public void distoryBitmap(){
        if(null!=bmb&&!bmb.isRecycled())
            bmb.recycle();
        }
        System.gc();
    }
{% endhighlight %}

我们还可以了解一下davlik虚拟机的内存管理相关的知识：

* [讲一讲Android davlik虚拟机的内存管理之内存分配](http://bbs.meizu.cn/thread-3796120-1-1.html)

那么怎么解决这个难题呢？针对这个问题，有几种方法来应对：

> 1. 调整虚拟机的堆栈大小；
2. 压缩或裁剪图片，减小Bitmap占用的内存；
3. 及时释放掉内存中的Bitmap。

对于上面的内容可以查看以下的链接：

* [Android加载大图片OOM异常解决](http://wenku.baidu.com/view/9a77922a7375a417866f8f4d.html)

事实上，没有100%的完美方案，调整虚拟机的堆栈大小，本质上对内存的限制还是在，所以还是有可能会出现OOM错误；而压缩或裁剪图片，也只是降低了出现OOM错误的概率。而对于释放Bitmap占用的内存这个方法，如果要及时释放掉内存中的一些Bitmap，那么当程序中刚好要用到这些释放掉的Bitmap时，又要重新加载进内存，这样肯定会影响了应用程序的运行速度的；并且，怎样才是及时呢？这个是很难判断的，并且哪些应该释放，哪些不应该释放呢？如果释放掉了一些还在使用的Bitmap，那么就有可能抛出异常：try to recycle a using bitmap。所以嘛！这又是一个很严峻的问题，可能也是android平台比较蛋疼的问题。据说在ios上面，系统就自带了一些对Bitmap等图片的缓存策略，开发者只需要做的是做好本地磁盘的缓存就足够了，莫非ios读取本地磁盘的内容是很快速的？但是我了解，在android上读取本地的图片还是需要一段比较长的时间的，所以从磁盘缓存图片来达到体验的流畅是坑爹的，不靠谱的。当然，我这里所说的要显示的页面的内容是比较丰富的那种页面哦。哈……唉唉……突然又有一种蛋蛋的疼……

所以，我们必须要折中考虑这个问题，全面的思考和总结，找出适用于当前的缓存策略来进行相应的处理。

对于Bitmap的缓存策略，一般有以下几种：

> 1. 使用本地磁盘缓存，将内存中暂时不需要的Bitmap以图片的形式储存在本地磁盘，当有需要的时候再从本地磁盘加载进来。
2. 使用软引用Bitmap，当内存不够时，软引用的Bitmap会自动被系统释放掉。
3. 使用内存的缓存，并且及时释放掉不需要的Bitmap。

（会涉及到LruCache、捕获异常、缓存通用的Bitmap对象）

其中，内存缓存这是必须的了，这是个人觉得！一般情况下，对于数据量少的ListView，普通的缓存策略就能解决，例如直接用内存来缓存，当不显示这个页面的时候就可以把这些数据和Bitmap释放掉。但是一个页面要显示很多的数据时，如果还这个页面还存在的情况下，不及时释放掉一些Bitmap内存，估计就会很快报出OOM错误了。

这里还有一篇总结得很好的文章，可以看看：

* [ANDROID内存管理](http://www.doc88.com/p-675124501108.html)

本人在一开始做“我家网”的android客户端的时候，显示动态页面的时候，我就直接把Bitmap缓存在内存中，只有当关闭了这个页面时或者关闭程序时，才连同程序一起释放内存。在“我家网”客户端上运行是没有问题的。因为要显示的图片数据比较少，一直都没有出现过OOM的错误。但是到了App分享这个项目时，我将我家网上面的哪些程序移植过来后，就经常会报OOM的错误。原因是App分享这个项目上的列表页面需要显示的数据量很多，成百上千条数据。普通的数据放到内存上，而那些图片数据Bitmap，也一直放到内存中了，因此就会报出了OOM的错误了。

所以，那时候我就开始思考，怎样的缓存策略才能使得应用程序运行流畅并且又不会报错呢？很不幸的是，没有100%的完美解决方案，如果谁有的话，希望可以提出来，本人感激万分啊！

下面是我研究之后得到的一些缓存的策略，大家可以参考参考，也可以改进改进……

{% highlight java linenos %}
    public Bitmap getPhoto(String photoUrl, boolean isSave, boolean isLoad,
            boolean toCompress) {
        Log.d(TAG, "getPhoto=" + photoUrl);
        String fileName = FileManage.getFileNameFromPath(photoUrl);
        Bitmap bitmap = null;
        BitmapCache bitmapCache = bitmapCacheBin.get(fileName);
        if (bitmapCache != null && bitmapCache.isCompressed == toCompress) {
            Log.d(TAG, "bitmapCacheBin");
            return bitmapCache.bitmap;
        }
        if (isLoad) {
            bitmap = loadSDCardCachePhoto(fileName, toCompress);
            if (bitmap != null) {
                Log.d(TAG, "loadSDCardCachePhoto");
                return bitmap;
            }
        }
        return loadHttpPhoto(photoUrl, fileName, isSave, toCompress);
    }
{% endhighlight %}

getPhoto()方法是获取图片资源的函数，返回值为Bitmap对象。

getPhoto()方法首先通过调用bitmapCacheBin.get(fileName)方法来检测是否已经在bitmapCacheBin缓存箱里面存在该Bitmap缓存。这个缓存箱是存在于内存的，其中，标志这个缓存的是这个fileName字符串，这个字符串是从getPhoto()方法的photoUrl参数截取出来的，并且是唯一的。如果在bitmapCacheBin中存在该缓存，那么就返回这个缓存的Bitmap对象。

如果不存在该缓存，那么就通过调用loadSDCardCachePhoto(fileName, toCompress)来检测本地磁盘是否存在该缓存文件，同样是由fileName字符串来唯一标志这个缓存文件。如果存在就从本地加载进内存，并且返回对应的Bitmap对象。事实上，加载进内存的时候，还需要做的是，把该Bitmap以内存缓存的形式放进bitmapCacheBin缓存箱里面，且看：

{% highlight java linenos %}
    public Bitmap loadSDCardCachePhoto(String fileName, boolean toCompress) {
        String filePath = null;
        if (toCompress) {
            filePath = CacheManage.getCacheFilePath("pic", fileName); // 缓存加载
        } else {
            filePath = CacheManage.getCacheFilePath("src_pic", fileName); // 缓存加载
        }
        BitmapCache bitmapCache = new BitmapCache(filePath, toCompress);
        if (bitmapCache.bitmap != null) {
            bitmapCacheBin.put(fileName, bitmapCache);
        }
        return bitmapCache.bitmap;
    }
{% endhighlight %}

其中，new BitmapCache(filePath, toCompress)生成一个Bitmap的缓存对象，这个对象是通过文件路径从本地磁盘寻找相应的文件文件并且加载进来的：

{% highlight java linenos %}
    public BitmapCache(String pathName, boolean toCompress) {
        Options options = new Options();
        bitmap = BitmapFactory.decodeFile(pathName, options);
        if (bitmap != null && toCompress) {
            bitmap = BitmapUtil.reduceImage(bitmap);
            isCompressed = true;
        }
        if (bitmap != null) {
            calculateByteCount(options.inPreferredConfig);
            this.pathName = pathName;
        }
    }
{% endhighlight %}

生成一个缓存对象之后，就通过bitmapCacheBin.put(fileName, bitmapCache)，把这个对象加进去bitmapCacheBin缓存箱里面去。

如果在本地磁盘上没有找到相应的缓存文件，那么就会通过调用loadHttpPhoto(photoUrl, fileName, isSave, toCompress)，从网络上去获取相应的图片，如果从网络上把图片获取回来之后，就会对返回来的图片进行缓存，并且返回相应的Bitmap，且看：

{% highlight java linenos %}
    public Bitmap loadHttpPhoto(String photoUrl, String fileName,
            boolean isSave, boolean toCompress) {
        InputStream in = HttpClient.getInputStream(URLString.getURLBase()
                + photoUrl);
        if (in != null) {
            BitmapCache bitmapCache = new BitmapCache(in, toCompress);
            if (isSave) {
                saveCachePhoto(bitmapCache.bitmap, fileName, toCompress);
            }
            bitmapCacheBin.put(fileName, bitmapCache);
            Log.d(TAG, "loadHttpPhoto");
            return bitmapCache.bitmap;
        }
        return null;
    }
{% endhighlight %}

loadHttpPhoto()方法首先会根据图片的url，调用HttpClient.getInputStream()方法返回http的输入流，然后根据这个输入流，通过new BitmapCache(in, toCompress)生成一个Bitmap的缓存对象，并且调用saveCachePhoto()把图片保存到到本地的磁盘缓存文件，接着再调用bitmapCacheBin.put(fileName, bitmapCache)，把Bitmap的缓存对象放进缓存箱，最后再返回Bitmap对象。

这就是本人用到的缓存的大概过程，个人觉得，一般缓存都需要这样去做，既要涉及到内存，又要涉及到本地的磁盘，到最后才是网络。但是还有很多细节需要注意的，那就是bitmapCacheBin缓存箱是怎样实现的，什么时候释放掉不用的Bitmap，以及什么时候再把需要的Bitmap加载进来。在这里本人遵循最近最少使用的原则，缩写是LRU，也就是：如果一个Bitmap对象最近不常使用的话会被排在缓存的后面，甚至不再保存，并且释放掉。在android4.0版本以上有一个类叫做LruCache，这个类实现了最近最少使用的算法，是经常用于缓存作用的类，详细可以查看这个类的源代码。你可以下载android-support-v4.jar这个jar包，这个包里面包含了LruCache这个类，你可以直接使用这个类；也可以使用jd-gui等查看jar文件的工具去查看LruCache这个类的实现，然后根据这个类的源代码来实现自己的一个LruCache类。下面是关于LruCache的使用的例子：

* [android 用LruCache读取大图片并缓存](http://androidkaifa.com/thread-52-1-1.html)

而本人所用的bitmapCacheBin缓存箱对象，是属于BitmapCacheBin类的一个实例，这个类就是参考了LruCache类，然后自己编写的。可以看看这个代码的部分实现：

{% highlight java linenos %}
    public class BitmapCacheBin {
        private final LinkedHashMap<String, BitmapCache> map;
        private int size;
        private int maxSize;
        private int putCount;
        private int createCount;
        private int evictionCount;
        private int hitCount;
        private int missCount;
    
        public BitmapCacheBin(int maxSize) {
            if (maxSize <= 0) {
                throw new IllegalArgumentException("maxSize <= 0");
            }
            this.maxSize = maxSize;
            this.map = new LinkedHashMap(0, 0.75F, true);
        }
    
        public final BitmapCache get(String key) {
            //……
        }
        
        public final BitmapCache put(String key, BitmapCache value) {
            //……
        }
        
        private void trimToSize(int maxSize) {
            //……
        }
        
        //……
    }
{% endhighlight %}

BitmapCacheBin类里面包括了一个带顺序的HashMap：LinkedHashMap，用于按一定顺序保存Bitmap缓存对象的。其中必须有get()、put()方法，用于取出和放进Bitmap缓存对象的；另外还需要一个trimToSize()方法来修改LinkedHashMap的内部结构，把最近最少使用的缓存对象，放在靠后的位置，或者直接释放掉。trimToSize()方法在put()方法，或者在某些情况下会调用，传入的参数是maxSize，表示这个缓存箱支持的最大的空间，这个空间与内存对应的，也与Bitmap的大小对应的，可以通过计算，互相转化。这个转化过程，可以参考以下的链接：

* [Android中图片占用内存的计算](http://blog.sina.com.cn/s/blog_4e60b09d01016133.html)
* [Android中图片占用内存的计算](http://meiyitianabc.blog.163.com/blog/static/105022127201271310159647/)

BitmapCacheBin类实现的主要的部分就是这些，大家可以在这个的基础上去改进，或者使用更好的策略来实现更好的缓存。例如我们也可以研究一下FastLRUCache类的实现，然后进行应用，这个可以查看下面的链接：

* [LRUCache和FastLRUCache实现分析](http://blog.sina.com.cn/s/blog_a577d05801016pil.html)

以及，可以把LRUCache与异步操作结合起来，使用Handler或者AsyncTask类来实现更有效率的内容，可以查看：

* [bitmap 缓存](http://wzrong-android.diandian.com/post/2012-04-20/19706052)

另外在缓存的过程中还有一些细节可以改善的，例如，从网上获取回来的图片，如果不是用于放大放小的浏览，也就是对于清晰度要求不高的话，我们可以对这个图片进行压缩，以致于减少在占用的内存。压缩的方法有两种，在网上也能找到这两种方法：

一种是通过Options的inSampleSize变量来进行压缩，详细可以看以下链接（或者直接百度：inSampleSize）：

* [解决Android加载图片时内存溢出的问题](http://263229365.iteye.com/blog/1562924)

另一种是通过Bitmap类的静态方法createScaledBitmap()来进行压缩。本人就是用这种方法来对图片进行压缩，因为这种压缩的效果比第一种的精确，因为第一种的压缩只能是按照整数倍去压缩，也就是只能压缩成1/2、1/3、1/4、1/8等等，而createScaledBitmap()可以按照一个固定的高度和宽度去压缩，这样会更精确。

回头看看新生成一个Bitmap的缓存对象的那段代码，

{% highlight java linenos %}
    public BitmapCache(String pathName, boolean toCompress) {
        Options options = new Options();
        bitmap = BitmapFactory.decodeFile(pathName, options);
        if (bitmap != null && toCompress) {
            bitmap = BitmapUtil.reduceImage(bitmap);
            isCompressed = true;
        }
        if (bitmap != null) {
            calculateByteCount(options.inPreferredConfig);
            this.pathName = pathName;
        }
    }
{% endhighlight %}

其中有一个地方调用了BitmapUtil.reduceImage(bitmap)方法，这个方法的实现是：

{% highlight java linenos %}
    public static Bitmap reduceImage(Bitmap bitmap, float scale) {
        final Bitmap oldBitmap = bitmap;
        System.out.println(bitmap.getWidth() + "," + bitmap.getHeight());
        bitmap = Bitmap.createScaledBitmap(oldBitmap, (int) (bitmap.getWidth()
                * scale + 1f), (int) (bitmap.getHeight() * scale + 1f), true);
        oldBitmap.recycle();
        return bitmap;
    }
    
    public static Bitmap reduceImage(Bitmap bitmap) {
        System.out.println("reduceImage");
        float scale = SIZE_WIDTH / (float) bitmap.getWidth();
        if (scale >= 1) {
            return bitmap;
        }
        return reduceImage(bitmap, scale);
    }
{% endhighlight %}

先调用第二个的reduceImage()方法，然后第二个的reduceImage()方法又调用第一个的reduceImage()方法，接着第一个的reduceImage()方法内部就会调用createScaledBitmap()方法把Bitmap进行压缩，生成新的Bitmap，然后释放掉原来的Bitmap。

另外要想继续完善的话，还可以考虑对生成Bitmap对象时进行异常捕获，然后对异常进行处理：

{% highlight java linenos %}
    try {
        // 实例化Bitmap
        bitmap = BitmapFactory.decodeFile(path);
    } catch (OutOfMemoryError e) {
        // 相应的处理，可以释放掉某些Bitmap缓存
    }
    if (bitmap == null) {
        // 如果实例化失败 返回默认的Bitmap对象
        return defaultBitmapMap;
    }
{% endhighlight %}

详细可以查看下面的链接：

* [Android应用开发中对Bitmap的内存优化](http://www.cdtarena.com/gpx/201210/5953.html)

另外为了使应用程序省去一些多余的工作，我们可以把一些常用的Bitmap对象常驻内存，例如一些默认头像图片或者出错图片就可以转化成Bitmap常驻于内存，当需要用的时候就直接引用就可以了。当然这些常驻内存的Bitmap不同滥用，也不能过多，不常用的Bitmap最好不要常驻于内存。

在说说软引用，软引用的使用其实很简单，下面给出一些实现的代码：

{% highlight java linenos %}
    public class CacheBin<T> {
        private final HashMap<String, SoftReference<T>> cacheMap = new HashMap<String, SoftReference<T>>();
    
        public void put(String key, T value) {
            cacheMap.put(key, new SoftReference<T>(value));
        }
    
        public T get(String key) {
            T value = null;
            SoftReference<T> reference = cacheMap.get(key);
            if (reference != null) {
                value = reference.get();
            }
            return value;
        }
    }
{% endhighlight %}

很明显，同样需要实现put()和get()方法。另外还有其他的软引用的例子，请查看：

* [SoftReference(软引用,弱引用)](http://wenku.baidu.com/view/9302b4b065ce050876321387.html)
* [通过软引用实现图片缓存，防止内存溢出](http://blog.csdn.net/zhou699/article/details/7301292)
* [通过软引用实现图片缓存，防止内存溢出](http://www.cnblogs.com/androidwsjisji/archive/2011/11/01/2231349.html)

我曾经反编译了QQ的android客户端，查看了部分的代码，发现它使用的缓存策略就是这种软引用，这样说的话QQ使用这个软引用还算是很成功的了。但是，我把这种策略用在了App分享的客户端，却没有如意的效果，还是有点莫名其妙的。不过google也建议最好不要用软引用来做缓存，至于为什么，我个人觉得可能是因为这种软引用不是很靠谱，可以说是很不稳定，很多时候需要用到的缓存，却被释放掉了；如果是这种情况的话，这样的缓存还真的是形同虚设啊。所以软引用要擅用啊……

最后就接近尾声了。但本人还意犹未尽，还很想和大家分享更多的东西。只是有些想法还实现得不靠谱，所以就不拿出来献丑了。其实我想说的是关于预加载的一些技术。这一般和浏览图片有关的一些想法。本人曾经自己突发奇想，编写了一个预加载的类，主要用于浏览相册的页面。这里预加载的意思，就是预先加载某一些位置上的bitmap，比如加载显示区域位置外向上几个以及向下几个位置的bitmap，这样做的话，当滚动相册的时候，就可以加快ImageView的显示了。刚好，今天找到一篇文章也介绍了这样的一些策略，有意者可以查看：

* [Bitmap内存溢出问题，解决策略](http://www.2cto.com/kf/201203/123773.html)

另外还可以尝试结合异步操作，并且记录需要设置的ImageView来改进，这本人还在尝试中。

{% highlight java linenos %}
    class PreLoadCaches{
        Bitmap []bitmap;
        ImageView []imageView;
        //……
    }
{% endhighlight %}

好了！散吧……