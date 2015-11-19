---
date: 2013-02-02 20:30:00+00:00
layout: post
title:  "异步操作"
categories: Android Utils
tags: Android 学习
excerpt: Android应用程序与服务器交互时，如果数据量比较大，或者网络速度缓慢，会导致
---

什么是异步？哈，你可以百度一下看看它的定义。不过我这里就只具体说Android异步加载服务器数据的相关内容。

Android应用程序与服务器交互时，如果数据量比较大，或者网络速度缓慢，会导致加载服务器数据的操作变得缓慢，甚至会出现阻塞等待的情况；如果这些与服务器交互的操作放到Android应用程序的UI线程上去执行，也就是放在Activity上直接执行，那么UI线程也相应的进行等待或阻塞，这时候Android手机的屏幕就会出现卡顿的现象，甚至出现不能响应用户操作的情况，这必定严重影响用户的操作体验。并且手机设备与网络服务器进行交互一般都是有延时的，也就是说，如果那些交互的操作放到UI线程上去执行，必定会有延时和阻塞。

为了解决这样的延时，我们必须要把交互的操作放到另一个线程上面去执行，当线程从服务器加载数据回来了，然后才在UI线程上显示数据。这就是异步操作。Android异步操作，一般是指在后台运行一段代码，进行某些处理，处理完之后，再提醒前台的线程，进行处理后的操作。而在这个处理的过程中不与前台的UI线程发生冲突和阻塞。

异步操作还有其他的好处！

异步操作的实现方法，可以参考以下的链接：

* [异步加载数据的三种实现](http://blog.csdn.net/sfshine/article/details/8050819)
* [Android异步处理一：使用Thread+Handler实现非UI线程更新UI界面](http://blog.csdn.net/mylzc/article/details/6736988)
* [Android异步处理二：使用AsyncTask异步更新UI界面](http://blog.csdn.net/mylzc/article/details/6772129)
* [Android异步处理三：Handler+Looper+MessageQueue深入详解](http://blog.csdn.net/mylzc/article/details/6771331)
* [Android异步处理四：AsyncTask的实现原理](http://blog.csdn.net/mylzc/article/details/6774131)

异步操作的实现方法主要还是Handler和AsyncTask，其中Handler据说会占有一定的开销，不过Handler实现异步操作，会显得更加灵活和方便。本人用Handler实现了一个异步操作的调度系统，以下这些内容是假设你已经对Handler的使用有一定的了解的情况下提出的。

这个调度系统，主要包括三个类，可以查看src文件夹下面的async文件夹里面的文件：

> * AsyncDispatcher.java
* AsyncTask.java
* AsyncListener.java

AsyncDispatcher是异步操作的任务调度者，里面包括任务队列和处理任务的线程；AsyncDispatcher的线程会从任务队列里面按顺序取出任务，然后交处理任务，处理完成后交给handler更新UI。

AsyncTask是表示的是异步操作的内容，其操作的内容重写在AsyncTask的handle()方法里面。

AsyncListener是异步处理的监听器，当异步处理完成并且成功后，就会执行onDone()方法；当异步处理失败之后（可能因为网络异常，或者内部的代码逻辑错误抛出异常），就会执行onException()方法；

怎样传递参数呢？且查看AsyncTask 源码：

{% highlight java linenos %}
    public abstract class AsyncTask implements Runnable {
        public int type = ExecuteThread.TYPE_NORMAL;
        public Object[] objects = null;
        public AsyncListener listener;
    
        AsyncTask() {
        }
    
        AsyncTask(int type, Object[] objects) {
            this.type = type;
            this.objects = objects;
        }
    
        public void asyncHandle(AsyncListener listener) {
            this.listener = listener;
            AsyncDispatcher.getInstance().put(type, this);
        }
    
        abstract void handle();
    
        public void run() {
            handle();
        }
    }
{% endhighlight %}

AsyncDo有一个参数public Object[] objects，当初始化一个AsyncDo对象时，传入objects参数，并且this.objects = objects;，故生成了AsyncTask对象之后就保存了需要传递的参数。

asyncHandle()方法的作用是生成一个AsyncTask任务对象，并把这个任务对象放进调度器AsyncDispatcher对象的任务队列里面；这是由AsyncDispatcher.getInstance().put()方法调用的，我们可以看看put()方法的代码：

{% highlight java linenos %}
    public synchronized void put(int type, Runnable task) {
        Object ticket = null;
        if (type == ExecuteThread.TYPE_NORMAL) {
            synchronized (taskQueue) {
                taskQueue.add(task);
                ticket = nomalTicket;
            }
        }
        //……
    }
{% endhighlight %}
    
很明显，通过taskQueue.add(task)方法把任务对象添加到taskQueue任务队列里面去。那么，这些任务对象是怎样取出来运行的呢？我们看看AsyncDispatcher的startThreads()方法是怎样的：

{% highlight java linenos %}
    public void startThreads() {
        Log.i(TAG, "startThreads()");
        if (isActive || threads == null) {
            return;
        }
        isActive = true;
        for (int i = 0; i < threads.size(); i++) {
            threads.get(i).setDaemon(true);
            threads.get(i).start();
        }
        //……
    }
{% endhighlight %}

startThreads()方法启动了调度器里面所有的线程，而这些线程是怎样定义和执行的呢？这些线程是ExecuteThread的对象，我们看看这个对象的run()方法就明了了：

{% highlight java linenos %}
    public void run() {
        isAlive = true;
        while (isAlive) {
            AsyncTask task = (AsyncTask) asyncDispatcher.poll(type);
            if (null != task) {
                try {
                    task.run();
                    Message msg = asyncDispatcher.handler.obtainMessage();
                    msg.obj = task;
                    msg.what = DOWN_CODE;
                    asyncDispatcher.handler.sendMessage(msg);
                } catch (Exception ex) {
                    ex.printStackTrace();
                    Message msg = asyncDispatcher.handler.obtainMessage();
                    msg.obj = task;
                    msg.what = EXCEPTION_CODE;
                    asyncDispatcher.handler.sendMessage(msg);
                }
            }
        }
    }
{% endhighlight %}

ExecuteThread线程对象通过asyncDispatcher的poll(type)方法从任务对象中取出任务对象，然后任务对象调用run()方法执行需要的处理（例如加载服务器数据），处理成功或者失败之后就调用handler.sendMessage(msg)方法把处理后的结果交给handler去处理。

那么handler是怎么处理的呢？看看handler对象的代码：

{% highlight java linenos %}
    Handler handler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            AsyncTask task = (AsyncTask) msg.obj;
            if (msg.what == EXCEPTION_CODE) {
                task.listener.onException();
            } else if (msg.what == DOWN_CODE) {
                task.listener.onDone();
            }
            // msg.what默认等于0
        }
    };
{% endhighlight %}

没错！执行成功后或失败后的善后处理就在这里执行，其中AsyncTask 对象是通过msg.obj变量来传递的。

但是还有一个疑问，那就是task.listener变量又是怎么回事？再看看AsyncTask 是怎么实现的：

{% highlight java linenos %}
    public abstract class AsyncTask implements Runnable {
        //……
        public AsyncListener listener;
        //……
        public void asyncHandle(AsyncListener listener) {
            this.listener = listener;
            AsyncDispatcher.getInstance().put(type, this);
        }
        //……
    }
{% endhighlight %}

asyncHandle()函数传入一个监听器对象AsyncListener给AsyncTask对象，然后AsyncTask对象把自己放进AsyncDispatcher调度器的任务队列里面。

且看一个使用这个异步操作的例子：

{% highlight java linenos %}
    // 加载数据的操作
    public void load(int paramInt) {
        System.out.println("load Something!");
        System.out.println("load Something!");
        System.out.println("load Something!");
        System.out.println("load Something!");
        System.out.println("load Something!");
        System.out.println("load Something!");
    }
    // 异步加载数据方法1
    public void asyncLoad(int paramInt) {
        new AsyncTask(TYPE_NORMAL, new Object[] { paramInt }) {
            @Override
            void handle() {
                load((Integer) objects[0]);
            }
        }.asyncHandle(new AsyncListener() {
            @Override
            public void onDone() {
                super.onDone();
                System.out.println("load onDone!, paramInt="
                        + (Integer) objects[0]);
            }
    
            @Override
            public void onException() {
                super.onException();
                System.out.println("load onException!, paramInt="
                        + (Integer) objects[0]);
            }
        });
    }
{% endhighlight %}

在asyncLoad()方法中首先生成了一个AsyncTask对象，并且重写了handle()方法，其中handle()方法调用了load()加载数据的方法；然后这个AsyncTask对象调用了asyncHandle()方法，并且传入一个AsyncListener监听器，并把加载完数据的操作重写在AsyncListener监听器的onDone()和onException()方法；AsyncTask的使用就是这样的了；

如果执行了asyncLoad()方法，那会有什么样的效果呢？

当前，前提是AsyncDispatcher调度器已经启动，AsyncDispatcher启动的方法是：

AsyncDispatcher.getInstance().startThreads();

AsyncDispatcher调度器启动后，执行asyncLoad()方法后，其整个处理过程是这样的：

> 1. new AsyncTask()，并传入所需的参数；
2. 调用asyncHandle()，并传入AsyncListener监听器对象给AsyncTask对象，在asyncHandle()方法里面，AsyncTask对象把自己放进AsyncDispatcher调度器的任务队列里面去；
3. 然后AsyncDispatcher的执行线程不断的从任务队列里面取出任务来执行，当取出asyncLoad()方法放进去的AsyncTask对象出来后，就调用AsyncTask对象的run()方法;
4. AsyncTask对象的run()方法会执行AsyncTask的抽象方法handle()，这个方法是在生成AsyncTask对象的时候重写了；
5. 重写的handle()方法，就会执行真正的处理函数load();
6. 如果load()方法执行成功之后就返回到handle()方法，handle()方法又返回到run()方法，然后接着继续执行线程接下来的代码；
7. 接下来的代码就是通过AsyncDispatcher的handler对象的sendMessage(msg)方法发送这个消息给这个handler对象；
8. 这个handler就会对这个消息进行相应的处理，输出“load onDone!, paramInt=1”；
9. 同理，如果load()方法执行失败之后，就会捕捉到异常，然后执行到catch块继续进行；
10. 然后handler对象的sendMessage(msg)方法发送这个消息给这个handler对象；
11. 这个handler就会对这个消息进行相应的处理，输出“load onException!, paramInt=1”；

这里的异步系统的AsyncTask与android自带的AsyncTask相比，有什么优缺点呢？之前本人看过一些文章，说用android自带的AsyncTask来做后台的异步处理比Handler做后台异步处理更轻量。是吗？那就打开android的AsyncTask类的源代码研究一下，发现，原来android的AsyncTask也是用Handler来做后台处理的，唉唉！兜兜转转，还是Handler，看来做异步处理的，肯定离不开Handler了。（严谨的说，Handler只是做善后的工作而已，真正的后台处理操作是放在后台线程里面进行的。）

不过有一点值得注意的是：在android的AsyncTask中，Handler是以AsyncTask类的静态变量存在的，也就是所有的AsyncTask对象共用一个Handler对象来进行善后的处理。Handler对象是一个有一定消耗的对象，当有多个异步操作同时需要进行的时候，如果每个操作都生成一个Handler对象进行异步操作的话，那这种叠加的消耗确实会不小；而AsyncTask来处理多个异步操作，其使用的只是一个Handler对象而已，所以相对来说，消耗会更小。

那么现在来证明我们的AsyncDispatcher调度器也有这样的优点。我们只允许生成一个AsyncDispatcher对象，是通过getInstance()这个静态方法获取AsyncDispatcher实例的，这个实例也就是调度器的唯一实例。为什么是唯一实例呢？我们可以看看getInstance()的实现代码就可以一目了然了，这里就不罗嗦了。

这个AsyncDispatcher实例，也就是AsyncDispatcher对象，里面只有唯一的一个Handler对象；AsyncDispatcher对象里面有若干的执行线程，执行线程从任务队列里面取出任务来执行，执行成功或失败后，都会把相应的消息发送给这个唯一的Handler对象进行善后的处理。Handler对象会按顺序从自己的消息队列里面取出消息然后进行相应的处理。

事实上，这里的AsyncDispatcher调度器的执行过程和android的AsyncTask的执行过程是很相似的。android的AsyncTask里面同样有类似于任务队列和有线程队列这样的东西。不同的是，这里的AsyncDispatcher调度器实现起来简单很多，并且还对AsyncTask任务进行分类处理。

这里的分类处理主要体现在AsyncTask的一个变量type，这个变量标志了这个任务的类型；当生成一个AsyncTask任务时，可以传入这type参数。当把这个任务放进任务队列时，就会根据这个type变量的值放到对应类型的任务队列里面；然后不同的任务队列就由这个任务队列对应的执行线程来执行。这样分类之后，有什么好处呢？

这样的分类主要是因为与服务器进行交互时，对服务器进行操作的种类不一样，以及加载的数据种类不一样。例如有些操作需要长时间才能返回数据，而有些是迅速的；如果那些长时间的操作阻碍了那些迅速的操作，这肯定会影响用户的体验。例如在一个页面中既要显示文字数据，又要显示图片，显示图片是一个长时间的操作和显示文字相对比较快，如果显示图片的操作是在显示文字的前面，这样就很有可能阻碍了显示文字的操作，使得这个页面一直都显示不完整，这肯定会影响了用户的使用体验。可能你又会说，那多启动几个线程就可以啦！是吗？那到底要启动多少个线程才满足要求呢？一个页面可能需要显示的内容很丰富，可能要加载很多的图片，或者需要加载很多其他的数据，那么难道要加载多少个数据就启动多少个线程？这很明显是不合理的啦。线程启动的数目一般在5个左右就好，或者更少，如果可以的话；毕竟线程是一个消耗不小的家伙呢！

另外，一般在一个android应用中，访问的服务器可能不止一个，不同的数据可能会分到不同的服务器上面，例如普通的数据放在数据库服务器中，上传的文件放在另一个服务器中，而图片相关的数据又放在另一个服务器中等等。这样，各个服务器之间的网络状态也可能会有不同，访问的速度也不一样。如果一个服务器的网络有些异样，例如储存图片的服务器出现网络异常，使得网络传输速度缓慢，如果在应用当中，刚好需要加载大量的图片，这时候，就会阻塞了其他的操作任务（例如加载普通数据的任务），这样肯定会对页面的显示速度造成一定的影响。

所以综合这些想法，个人觉得对这些任务进行分类是有必要的。因为这样做，就可以使得不同的数据服务器不会互相影响，也能使得不同种类的数据不会互相阻碍。例如，本AsyncDispatcher系统中，定义了几个类型的执行线程，用来执行相应类型的任务，如下：

{% highlight java linenos %}
    public static class ExecuteThread extends Thread {
        public static final int TYPE_NORMAL = 0;
        public static final int TYPE_GET_PHOTO = 1;
        public static final int TYPE_GET_BIG_PHOTO = 2;
        public static final int TYPE_GET_NOTI_MSG = 3;
        int type = -1;
        AsyncDispatcher asyncDispatcher;
        //……
{% endhighlight %}

分别是普通数据的类型、获取小图片的类型、获取大图片的类型、获取通知（推送）消息的类型等等。

分析到这里，差不多了…就这样吧！散……