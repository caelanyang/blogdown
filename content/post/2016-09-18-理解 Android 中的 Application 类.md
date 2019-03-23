---
title: 理解 Android 中的 Application 类
author: Jiacheng
date: '2016-09-18'
slug: '2016-09-18'
categories:
  - Technology
tags: 
  - Android
  - Translate
---

## 概述
在 Android 中，Application 类是 APP 的基础类，它包含了 APP 中其他的所有组件，如 Activity 和 Service 等。Application 或 Application 的子类在 APP 被创建的时候会先于其他的类被实例化。

这个类主要用于在第一个 Activity 显示之前初始化应用的全局状态。需要注意的是，要谨慎使用自定义的 Application 类，因为通常情况下是没有必要的。

## 自定义 Application 类


在很多应用中都是不需要直接使用 Application 类的。然而，自定义的 Application 类有以下几个可被接受的的用途：

* 运行需要在第一个 Activity 创建之前执行的任务
* 需要在所有组件之间共享的全局初始化（比如崩溃报告，持久化）
* 访问静态不可变数据的静态方法，比如共享的网络客户端对象

注意不要在 Application 对象中存储可变的共享数据，因为这样的数据可能会随时消失或变为无效。相反的，这时候应该使用持久化策略存储可变的共享数据，比如 File，SharedPreferences，或 SQLite 等持久化方法。

## 如何定义 Application 类

要自定义 Application 类，首先创建一个继承 android.app.Application 的新类。如下：

```java
import android.app.Application;

public class MyApplication extends Application {
        // Called when the application is starting, before any other application objects have been created.
        // Overriding this method is totally optional!
	@Override
	public void onCreate() {
	    super.onCreate();
            // Required initialization logic here!
	}

        // Called by the system when the device configuration changes while your component is running.
        // Overriding this method is totally optional!
	@Override
	public void onConfigurationChanged(Configuration newConfig) {
	    super.onConfigurationChanged(newConfig);
	}

        // This is called when the overall system is running low on memory, 
        // and would like actively running processes to tighten their belts.
        // Overriding this method is totally optional!
	@Override
	public void onLowMemory() {
	    super.onLowMemory();
	}
}
```

然后修改 AndroidManifest.xml 中 Application 标签的 `android:name` 属性，如下：

```java
<application 
   android:name=".MyApplication"
   android:icon="@drawable/icon" 
   android:label="@string/app_name" 
   ...>
```

这样就完成了自定义 Application 类的操作。

## 限制和警告

在 APP 中，总是会有一些数据和信息需要在很多地方使用。也许是 session token，也许是一次复杂计算的结果，等等。有人倾向于直接使用 Application 的实例来存储上述的数据或信息，这样可以避免在活动之间传递对象或保存在持久存储中等方法带来的开销，这听起来是很诱人的。

但是，不应该将可变的实例数据保存在 Application 中，如果这样做了，应用程序将不可避免地在某个时刻发生 `NullPointerException` 崩溃。因为 Application 对象无法保证永远保留在内存中，它将会被杀死。普遍的想法是 APP 不会从头开始重新启动。与之相反的是，Android 将会创建一个新的 Application 对象并启动用户上次停留的 Activity，这会造成 Application 上一次未被杀死的错觉。

所以应该通过以下方式之一存储共享数据：

* 通过 Intent 明确地将数据传递给 Activity。
* 使用持久化策略将数据保存到磁盘中。

**总之**：将数据存储在 Application 对象中容易出错，并可能导致应用程序崩溃。对于以后真正需要用到的全局数据，最好将其存储在磁盘上，或者在 Intent 的 Extra 中明确地传递给 Activity。

> 点击 [原文链接](https://guides.codepath.com/android/Understanding-the-Android-Application-Class#limitations-and-warnings) 查看英文原文。

