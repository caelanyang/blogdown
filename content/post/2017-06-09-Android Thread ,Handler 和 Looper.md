---
title: Android Thread，Handler 和 Looper
author: Jiacheng
date: '2017-06-09'
slug: '2017-06-09'
categories:
  - Technology
tags: 
  - Android
---


## 概述
所有的 APP 启动的时候都会运行在同一个主线程中，也就是用户界面（UI）线程。
在同一进程下运行的所有组件都在主线程中运行。

但是在 UI 线程中智行耗时比较长的操作就会阻塞 UI 线程，而且，如果 UI 线程被阻塞超过五秒钟就会引起 ANR 。

耗时比较长的典型操作如数据库事物，文件下载，复杂的计算等，应该在后台线程获工作线程中完成。

> Android 中不能在后台线程或工作线程中直接与UI元素相互作用，于是使用 Handler 在 UI 线程和后台线程或工作线程之间进行通信。

如果直接在 UI 线程之外的别的线程直接刷新 UI 组件，便会抛出『java.lang.RuntimeException:Cannot update UI from another thread』。

## MessageQueue
MessageQueue 是 Android 维持的一个任务队列，这些任务会按顺序一个接一个执行。<!-- more -->
在 Android 中 Message 不能被直接加入到 MessageQueue 中，只能使用 Handler 的对象添加。

## Handler 
Handler 负责产生 Message ，发送 Message 到 MessageQueue，以及处理从 MessageQueue 中弹出的 Message。每一个 Handler 都必须与一个 Looper 绑定。

## Looper 
Looper 轮询 MessageQueue 中的下一个 Message，每当遇到一个 Message，便把 Message 传递给对应的 Handler 处理。
Android 中，除了 UI 线程以外，普通的线程默认情况下是不自带 Looper 的。想要通过 UI 线程与子线程通信需要在子线程内创建一个 Looper。开启 Looper 分三步走：

1. 判断是否已有 Looper 并 `Looper.prepare()`
2. 做一些准备工作(如实例化 Handler 等)
3. 调用 Looper.loop()，线程进入阻塞态，等待处理 Message

典型代码如下：

~~~java
  class LooperThread extends Thread {
      public Handler mHandler;

      public void run() {
          Looper.prepare();

          mHandler = new Handler() {
              public void handleMessage(Message msg) {
                  // process incoming messages here
              }
          };

          Looper.loop();
      }
  }
  
~~~
  


> 注意: 一个线程只能关联一个 Looper，于是也只能关联一个 MessageQueue。但是一个 Looper 可以关联多个 Handler，只是这些 Handler 向同一个 MessageQueue 中插入 Message。
> 
> 如果多个线程尝试向一个特定的 Looper 发送 Message，所有这些线程将会被顺序处理。

如果尝试为同一个线程建立第二个 Looper，便会导致『java.lang.RuntimeException:Only one Looper may be created per thread』

**Android 是怎么检查线程是否已经关联一个 Looper 的 ？**

调用 `Looper.prepare() `创建 Looper 时，会首先检查当前线程的 ThreadLocal，确保还没有关联 Looper ，只有检查通过之后，才会创建新的 Looper 并保存在 ThreadLocal。


