---
date: 2014-02-14 20:30:00+00:00
layout: post
title:  "android实时语音方案"
categories: Android
tags: Android 实例 speex speech
excerpt: 客户端需要使用AudioRecord、AudioTrack、Thread、UDP/TCP、Speex等知识和技术。
---

![img](/assets/2014-02-14-android-speex-speech.png)
<p></p>

客户端（Android）
------

客户端需要使用AudioRecord、AudioTrack、Thread、UDP/TCP、Speex等知识和技术。

* 说话（录制）端：

> 1. 新线程Thread创建AudioRecord对象并启动录制，然后循环调用AudioRecord.read()方法，获得PCM数据；
2. 再使用speex对PCM数据进行编码和压缩，生成spx数据包；
3. 再将spx数据包压到另外一个线程的数据处理队列中；
4. 在另外一个线程的run里面，循环从队列里面取出一个spx数据包，然后使用UDP或TCP打包传输到服务端；服务端会把这些数据包转发到另一个客户端。

* 收听（播放）端：

> 1. 新线程使用UDP或TCP接收服务端转发的数据包；
2. 对接收到的数据包进行解包，生成spx数据包，并压到另外一个线程的数据处理队列中；
3. 在另外一个线程的run里面循环从队列里面取出一个spx数据包并使用speex对spx数据包进行解码，生成PCM数据；
4. 调用audioTrack.write()方法把PCM数据写到AudioTrack对象的缓冲区中，这样子AudioTrack对象就能对PCM数据进行播放；
<p></p>


服务端（语音转发服务器）：
------

> 1. 使用UDP或TCP协议对数据包进行转发，服务端支持多客户端的链接和消息的收发。
2. 这个服务端是专门服务于实时语音的，与同一个app中的其他后台服务分离。
3. 客户端呼叫和接收呼叫，这些操作都不属于这个语音转发服务器，而属于app中原来聊天的服务器管理的；
4. 对端一旦接受了呼叫，那么这两个将要对话的客户端就会在这个语音转发服务器都建立了连接，这时候，这两个客户端就可以通过这个服务端的转发，实时地发送和接收语音了。

现在客户端已经实现了一个一边录制和speex编码，一边speex解码和播放的的demo。

* [Recorder](https://github.com/Janseon/Recorder)