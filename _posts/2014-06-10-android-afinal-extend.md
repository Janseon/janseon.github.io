---
date: 2014-06-12 20:30:00+00:00
layout: post
title:  "Afinal扩展"
categories: Android Utils
tags: Android 学习 Afinal
excerpt: Afinal是什么，到github看看吧
---

Afinal是什么，到github看看吧：[Afinal](https://github.com/yangfuhai/afinal)
<p></p>

该文章中，为Afinal扩展了什么，可以到我的github看看：[Afinal扩展](https://github.com/Janseon/afinal)
<p></p>

1、添加publishStart
------

添加这个方法，主要是为了给http相关的操作进行扩展。

通过这个扩展，可以使得在http请求进行之前，先加载之前曾经请求的http缓存。

在下面的类中，进行修改：

{% highlight java linenos %}
   net.tsz.afinal.http.HttpHandler
{% endhighlight %}

修改为：

{% highlight java linenos %}
   @Override
   protected Object doInBackground(Object... params) {
      if (params != null && params.length == 3) {
         targetUrl = String.valueOf(params[1]);
         isResume = (Boolean) params[2];
      }
      try {
         publishStart();
         makeRequestWithRetries((HttpUriRequest) params[0]);
   
      } catch (IOException e) {
         String msg = e.getMessage();
         if (msg == null) {
            msg = e.toString();
         }
         publishProgress(UPDATE_FAILURE, e, 0, msg); // 结束
      }
   
      return null;
   }
{% endhighlight %}

因为publicProgress方法是final方法，不能被重写；

所以封装一下publicProgress(UPDATE_START)方法的调用，那么pubicStart()方法就可以被重写了。

{% highlight java linenos %}
   protected void publishStart() {
      publishProgress(UPDATE_START); // 开始
   }
{% endhighlight %}

使用时，继承HttpHandler，重写如下，进行缓存的加载：

{% highlight java linenos %}
   @Override
   protected void publishStart() {
      super.publishStart();
      if (ajaxCallBack instanceof AsyncCallBack) {
         final AsyncCallBack asyncCallBack = ((AsyncCallBack) ajaxCallBack);
         if (asyncCallBack.toLoad == 1) {
            String urlId = getUrlId();
            final Response response = Response.load(urlId);
            if (response != null) {
               parse(response, null);
               AppContext.runUI(new Runnable() {
                  @Override
                  public void run() {
                     asyncCallBack.onLoadResponse(response);
                  }
               });
            }
         }
      }
   }
{% endhighlight %}
<p></p>

2、加载图片时，显示进度
------

显示如下加载进度：

![img](/assets/2014-06-10-android-afinal-extend.png)

修改如下类：

{% highlight java linenos %}
   net.tsz.afinal.FinalBitmap
{% endhighlight %}

修改如下：

{% highlight java linenos %}
   public static abstract class AsyncDrawable extends Drawable {
      private final BitmapLoadAndDisplayTask mTask;
      public Drawable mDrawable;
   
      /**
       * 画笔对象的引用
       */
      private Paint paint;
   
      /**
       * 圆环的颜色
       */
      private int roundColor;
   
      /**
       * 圆环进度的颜色
       */
      private int roundProgressColor;
   
      /**
       * 中间进度百分比的字符串的颜色
       */
      private int textColor;
   
      /**
       * 中间进度百分比的字符串的字体
       */
      private float textSize;
   
      /**
       * 圆环的宽度
       */
      private float roundWidth;
      /**
       * 圆环的半径
       */
      private float roundRadius;
   
      /**
       * 当前进度
       */
      private int progress;
   
      public AsyncDrawable(Context context, BitmapLoadAndDisplayTask bitmapWorkerTask) {
         mTask = bitmapWorkerTask;
         paint = new Paint(Paint.ANTI_ALIAS_FLAG);
         paint.setAntiAlias(true);
         DisplayMetrics metrics = context.getResources().getDisplayMetrics();
         textSize = TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_SP, 12, metrics);
         roundWidth = TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP, 3, metrics);
         roundRadius = TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP, 15, metrics);
         roundColor = Color.WHITE;
         roundProgressColor = Color.BLUE;
         textColor = Color.WHITE;
         progress = -1;
      }
   
      public BitmapLoadAndDisplayTask getBitmapWorkerTask() {
         return mTask;
      }
   
      @Override
      public void draw(Canvas canvas) {
         Rect bounds = getBounds();
         if (mDrawable != null) {
            mDrawable.setBounds(bounds);
            mDrawable.draw(canvas);
         }
         if (progress == -1) {
            return;
         }
         // if (bounds.width() / 2 < roundRadius || bounds.height() / 2 <
         // roundRadius) {
         // return;
         // }
   
         /**
          * 画最外层的大圆环
          */
         int centreX = bounds.centerX(); // 获取圆心的x坐标
         int centreY = bounds.centerY(); // 获取圆心的x坐标
   
         paint.setColor(roundColor); // 设置圆环的颜色
         paint.setStyle(Paint.Style.STROKE); // 设置空心
         paint.setStrokeWidth(roundWidth); // 设置圆环的宽度
         canvas.drawCircle(centreX, centreY, roundRadius, paint); // 画出圆环
   
         /**
          * 画圆弧 ，画圆环的进度
          */
         paint.setStrokeWidth(roundWidth); // 设置圆环的宽度
         paint.setColor(roundProgressColor); // 设置进度的颜色
         RectF oval = new RectF(centreX - roundRadius, centreY - roundRadius, centreX + roundRadius, centreY + roundRadius); // 用于定义的圆弧的形状和大小的界限
         paint.setStyle(Paint.Style.STROKE);
         canvas.drawArc(oval, 0, 360 * progress / 100, false, paint); // 根据进度画圆弧
   
         /**
          * 画进度百分比
          */
         paint.setStrokeWidth(0);
         paint.setColor(textColor);
         paint.setTextSize(textSize);
         // paint.setTypeface(Typeface.DEFAULT_BOLD); // 设置字体
         String progressText = progress + "%";
         float textWidth = paint.measureText(progressText); // 测量字体宽度，我们需要根据字体的宽度设置在圆环中间
         canvas.drawText(progressText, centreX - textWidth / 2, centreY + textSize / 2, paint); // 画出进度百分比
      }
   
      public void setProgress(int progress) {
         this.progress = progress;
         invalidateSelf();
      }
   
      @Override
      public int getOpacity() {
         return 0;
      }
   
      @Override
      public void setAlpha(int arg0) {
      }
   
      @Override
      public void setColorFilter(ColorFilter arg0) {
      }
   };
{% endhighlight %}

以上类，详细的实现可以看draw方法会progress进行绘制的操作

{% highlight java linenos %}
   public static class AsyncColorDrawable extends AsyncDrawable {
      public static int color = 0xaabbbbbb;
   
      public AsyncColorDrawable(Context context, BitmapLoadAndDisplayTask bitmapWorkerTask) {
         this(context, bitmapWorkerTask, color);
      }
   
      public AsyncColorDrawable(Context context, BitmapLoadAndDisplayTask bitmapWorkerTask, float radius) {
         super(context, bitmapWorkerTask);
         if (radius == -1) {
            ShapeDrawable shapeDrawable = new ShapeDrawable(new OvalShape());
            shapeDrawable.getPaint().setColor(color);
            mDrawable = shapeDrawable;
         } else if (radius == 0) {
            mDrawable = new ColorDrawable(color);
         } else if (radius > 0) {
            float[] outerR = new float[] { radius, radius, radius, radius, radius, radius, radius, radius };
            RoundRectShape roundRectShape = new RoundRectShape(outerR, null, null);
            ShapeDrawable shapeDrawable = new ShapeDrawable(roundRectShape);
            shapeDrawable.getPaint().setColor(color);
            mDrawable = shapeDrawable;
         }
      }
   
      public AsyncColorDrawable(Context context, BitmapLoadAndDisplayTask bitmapWorkerTask, int color) {
         super(context, bitmapWorkerTask);
         mDrawable = new ColorDrawable(color);
      }
   }
{% endhighlight %}

以上只是简单的一个继承，实现了一个背景是灰色的默认显示，如加载前的默认显示和加载失败之后的显示。

{% highlight java linenos %}
   private class BitmapLoadAndDisplayTask extends AsyncTask<Object, Integer, Bitmap> {
      
      @Override
      protected Bitmap doInBackground(Object... params) {
         dataString = (String) params[0];
         Bitmap bitmap = null;
   
         synchronized (mPauseWorkLock) {
            while (mPauseWork && !isCancelled()) {
               try {
                  mPauseWorkLock.wait();
               } catch (InterruptedException e) {
               }
            }
         }
         
         if (bitmap == null && !isCancelled() && !mExitTasksEarly) {
            if (displayConfig.getLoading()) {
               bitmap = processBitmap(dataString, displayConfig, new ProgressCallBack() {
                  @Override
                  public void callBack(long count, long current) {
                     int progress = (int) (current * 100 / count);
                     publishProgress(progress);
                  }
               });
            } else {
               bitmap = processBitmap(dataString, displayConfig, null);
            }
         }
   
         if (bitmap != null) {
            mImageCache.addToMemoryCache(dataString, bitmap);
         }
         return bitmap;
      }
   
      @Override
      protected void onProgressUpdate(Integer... values) {
         int progress = values[0];
         final View[] imageViews = getAttachedImageView();
         for (View imageView : imageViews) {
            if (imageView instanceof ImageView) {
               Drawable drawable = ((ImageView) imageView).getDrawable();
               if (drawable instanceof AsyncDrawable) {
                  ((AsyncDrawable) drawable).setProgress(progress);
               }
            }
         }
      }
   }
{% endhighlight %}

以上主要是看onProgressUpdate的更新方法。

那么还要修改http请求，使得加载图片有进度的显示

修改如下类：

{% highlight java linenos %}
   net.tsz.afinal.bitmap.download.SimpleDownloader
{% endhighlight %}

{% highlight java linenos %}
   private byte[] getFromHttp(String urlString, ProgressCallBack callBack) {
      HttpURLConnection urlConnection = null;
      BufferedOutputStream out = null;
      FlushedInputStream in = null;
   
      try {
         final URL url = new URL(urlString);
         urlConnection = (HttpURLConnection) url.openConnection();
         in = new FlushedInputStream(new BufferedInputStream(urlConnection.getInputStream(), IO_BUFFER_SIZE));
         ByteArrayOutputStream baos = new ByteArrayOutputStream();
         int b;
         if (callBack == null) {
            while ((b = in.read()) != -1) {
               baos.write(b);
            }
         } else {
            int len = urlConnection.getContentLength();
            if (len == 0) {
               return null;
            }
            final int progressSize;
            if (len < PEOGRESS_SIZE * 5) {
               progressSize = len / 5;
            } else {
               progressSize = PEOGRESS_SIZE;
            }
            long current = 0;
            callBack.callBack(len, current);
            int callLen = 0;
            while ((b = in.read()) != -1) {
               baos.write(b);
               callLen++;
               if (current == len || callLen > progressSize) {
                  // SystemClock.sleep(500);
                  current += callLen;
                  callBack.callBack(len, current);
                  callLen = 0;
               }
            }
         }
         return baos.toByteArray();
      } catch (final IOException e) {
         Log.e(TAG, "Error in downloadBitmap - " + urlString + " : " + e);
      } finally {
         if (urlConnection != null) {
            urlConnection.disconnect();
         }
         try {
            if (out != null) {
               out.close();
            }
            if (in != null) {
               in.close();
            }
         } catch (final IOException e) {
         }
      }
      return null;
   }
{% endhighlight %}
<p></p>

3、同一个链接，避免多次http请求
------

doDisplay方法中：

修改如下，主要是封装了一个createAsyncDrawable方法

{% highlight java linenos %}
   if (bitmap != null) {
      if (imageView instanceof ImageView) {
         ((ImageView) imageView).setImageBitmap(bitmap);
      } else {
         imageView.setBackgroundDrawable(new BitmapDrawable(bitmap));
      }
   }
   else {
      createAsyncDrawable(imageView, uri, displayConfig);
   }
{% endhighlight %}

主要实现方式是：维持一个对AsyncDrawable对象的集合的缓存引用：mAsyncDrawablesCache

{% highlight java linenos %}
   private AsyncDrawable createAsyncDrawable(View imageView, String uri, BitmapDisplayConfig config) {
      AsyncDrawable asyncDrawable = mAsyncDrawablesCache.get(uri);
      if (asyncDrawable != null) {
         setImageDrawable(imageView, asyncDrawable);
         BitmapLoadAndDisplayTask task = asyncDrawable.getBitmapWorkerTask();
         task.putImageView(imageView);
         Log.i("setImage", "putImageView,imageView,url=" + task + "," + imageView + "," + uri);
         return asyncDrawable;
      }
      BitmapLoadAndDisplayTask task = new BitmapLoadAndDisplayTask(imageView, config);
      Log.i("setImage", "BitmapLoadAndDisplayTask,imageView,url=" + task + "," + imageView + "," + uri);
      Bitmap bitmap = config.getLoadingBitmap();
      asyncDrawable = bitmap == null ? new AsyncColorDrawable(imageView.getContext(), task, config.getLoadingRadius()) : new AsyncBitmapDrawable(
            imageView.getContext(), task, bitmap);
      asyncDrawable.roundProgressColor = config.getLoadingColor();
      setImageDrawable(imageView, asyncDrawable);
      mAsyncDrawablesCache.put(uri, asyncDrawable);
      task.executeOnExecutor(bitmapLoadAndDisplayExecutor, uri);
      return asyncDrawable;
   }
{% endhighlight %}

{% highlight java linenos %}
   private static final SoftMemoryCache<AsyncDrawable> mAsyncDrawablesCache = new SoftMemoryCache<AsyncDrawable>();
{% endhighlight %}

BitmapLoadAndDisplayTask 中又维持一个View的列表引用：

{% highlight java linenos %}
   private class BitmapLoadAndDisplayTask extends AsyncTask<Object, Integer, Bitmap> {
      private String dataString;
      private ArrayList<WeakReference<View>> imageViewReferences = new ArrayList<WeakReference<View>>();
      private final BitmapDisplayConfig displayConfig;
   
      public BitmapLoadAndDisplayTask(View imageView, BitmapDisplayConfig config) {
         // imageViewReference = new WeakReference<View>(imageView);
         imageViewReferences.add(new WeakReference<View>(imageView));
         displayConfig = config;
      }
   
      public void putImageView(View imageView) {
         imageViewReferences.add(new WeakReference<View>(imageView));
      }
       //省略...
      @Override
      protected void onProgressUpdate(Integer... values) {
         int progress = values[0];
         final View[] imageViews = getAttachedImageView();
         for (View imageView : imageViews) {
            if (imageView instanceof ImageView) {
               Drawable drawable = ((ImageView) imageView).getDrawable();
               if (drawable instanceof AsyncDrawable) {
                  ((AsyncDrawable) drawable).setProgress(progress);
               }
            }
         }
      }
   
      @Override
      protected void onPostExecute(Bitmap bitmap) {
         mAsyncDrawablesCache.remove(dataString);
         if (isCancelled() || mExitTasksEarly) {
            bitmap = null;
         }
         // 判断线程和当前的imageview是否是匹配
   
         Log.i("setImage", "onPostExecute");
         // final View imageView = getAttachedImageView();
   
         final View[] imageViews = getAttachedImageView();
         for (View imageView : imageViews) {
            if (bitmap != null && imageView != null) {
               mConfig.displayer.loadCompletedisplay(imageView, bitmap, displayConfig);
            } else if (bitmap == null && imageView != null) {
               mConfig.displayer.loadFailDisplay(imageView, displayConfig.getLoadfailBitmap());
            }
         }
      }
   
      // 获取线程匹配的imageView,防止出现闪动的现象
      private View[] getAttachedImageView() {
         View[] views = new View[imageViewReferences.size()];
         int i = 0;
         for (WeakReference<View> imageViewReference : imageViewReferences) {
            final View imageView = imageViewReference.get();
            if (imageView != null) {
               final BitmapLoadAndDisplayTask bitmapWorkerTask = getBitmapTaskFromImageView(imageView);
               Log.i("setImage", "getAttachedImageView,imageView,url=" + this + "," + imageView + "," + dataString);
               if (this == bitmapWorkerTask) {
                  views[i++] = imageView;
               }
            }
         }
         return views;
      }
   }
{% endhighlight %}
<p></p>

4、三级缓存
------

哪三级缓存：

> 1. 根据url和图片大小，构造key，获取内存中的bitmap；如果为空，则 2；
2. 根据 1 中构造的key，获取磁盘中的bitmap data缓存；如果为空，则 3；
3. 根据url，获取磁盘中的bitmap data缓存；如果不为空，则 4 ；如果为空则加载http 图片，如 5 ；
4. 根据加载的bitmap data，裁剪成需要的bitmap大小，并且添加进磁盘和内存缓存；
5. 根据url进行http请求，返回bitmap data，把bitmap data储存进磁盘缓存，并且裁剪成需要的大小的bitmap data，然后分别添加进磁盘缓存和bitmap缓存。

修改如下类：

{% highlight java linenos %}
   net.tsz.afinal.bitmap.core.BitmapProcess
{% endhighlight %}

主要方法：http请求成功之后：调用mCache.addToDiskCache()

{% highlight java linenos %}
   public Bitmap getBitmap(String url, BitmapDisplayConfig config, ProgressCallBack callBack) {
   
      Bitmap bitmap = getFromDisk(url, config);
   
      if (bitmap == null) {
         byte[] data = mDownloader.download(url, callBack);
         if (data != null && data.length > 0) {
            mCache.addToDiskCache(url, data);
            if (config != null)
               bitmap = BitmapDecoder.decodeSampledBitmapFromByteArray(data, 0, data.length, config.getBitmapWidth(), config.getBitmapHeight());
            else
               bitmap = BitmapFactory.decodeByteArray(data, 0, data.length);
         }
      }
      return bitmap;
   }
{% endhighlight %}

主要方法：从磁盘缓存加载url对应的图片之后，进行裁剪，调用：BitmapDecoder.decodeSampledBitmapFromByteArray()方法：

{% highlight java linenos %}
   public Bitmap getFromDisk(String key, BitmapDisplayConfig config) {
      BytesBuffer buffer = sMicroThumbBufferPool.get();
      Bitmap b = null;
      try {
         boolean found = mCache.getImageData(key, buffer);
         if (found && buffer.length - buffer.offset > 0) {
            if (config != null) {
               b = BitmapDecoder.decodeSampledBitmapFromByteArray(buffer.data, buffer.offset, buffer.length, config.getBitmapWidth(),
                     config.getBitmapHeight());
            } else {
               b = BitmapFactory.decodeByteArray(buffer.data, buffer.offset, buffer.length);
            }
         }
      } finally {
         sMicroThumbBufferPool.recycle(buffer);
      }
      return b;
   }
{% endhighlight %}

在以下内部类进行修改：

{% highlight java linenos %}
   net.tsz.afinal.FinalBitmap.BitmapLoadAndDisplayTask
{% endhighlight %}

主要方法：把裁剪后的图片添加到内存缓存：

{% highlight java linenos %}
   @Override
   protected Bitmap doInBackground(Object... params) {
      dataString = (String) params[0];
      Bitmap bitmap = null;
   
      synchronized (mPauseWorkLock) {
         while (mPauseWork && !isCancelled()) {
            try {
               mPauseWorkLock.wait();
            } catch (InterruptedException e) {
            }
         }
      }
      if (bitmap == null && !isCancelled() && !mExitTasksEarly) {
         if (displayConfig.getLoading()) {
            bitmap = processBitmap(dataString, displayConfig, new ProgressCallBack() {
               @Override
               public void callBack(long count, long current) {
                  int progress = (int) (current * 100 / count);
                  publishProgress(progress);
               }
            });
         } else {
            bitmap = processBitmap(dataString, displayConfig, null);
         }
      }
   
      if (bitmap != null) {
         mImageCache.addToMemoryCache(dataString, bitmap);
      }
      return bitmap;
   }
{% endhighlight %}