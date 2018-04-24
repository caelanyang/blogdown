---
title: RecycleView Adapter 的解耦实现
author: 杨嘉程
date: '2018-04-18'
slug: recycleview-adapter-decouple
categories:
  - Technology
tags:
  - Android
  - RecycleView
  - Adapter
  - 解耦
---

前段时间由于 APP 业务上的原因，写了一个相对比较复杂的 RecycleView 的 Adapter，这样就导致 adapter 里面集中了大量的代码。在其中的 `onCreateViewHolder()`，`onBindViewHolder()`,以及 `getItemViewType` 这三个方法里面堆积了大量的 if-else 或者 switch 语句，特别是 RecycleView 的 item 类型比较多的时候。当增加或删除 Item类型的时候，这三个方法基本都要同时修改，修改其中的逻辑分支。

另外，当添加一些 header 或与数据 List 无关的 Item 的类型的时候，往往有需要计算数据的偏移（offset），如果这种类型的 Item 个数 不固定的话，还要继续维护一个可变的 offset 计算。

以及，可能还需要给不同的 Item 创建不同的 ViewHolder。

这时候，代码写起来麻烦，当业务逻辑发生变动，改起来也很麻烦，牵一发而动全身。遇到 bug 又要一遍遍的检查逻辑。代码可读性差。

之所以会是这个样子，是因为代码的耦合度太高了。Adapter 的官方文档对它的注释是『dapters provide a binding from an app-specific data set to views that are displayed within a RecyclerView』，大意是 adapter 提供了 RecycleView 中指定数据集到 View 视图的绑定。

实际使用中也确实如此，那么问题就出在这里，Adapter 作为一个适配器，不但需要维护所需的数据集，还要创建所需的视图，并且将两者进行对应绑定。当 Item 的类型比较单一的时候，还没什么，但是当一个 RecycleView 所包含的 Item 的类型增加的时候，就会出现前面所说的那些问题。

于是，我就尝试着把 Adapter 进行解耦，而且也仅仅只是解耦:smile:。

回到前面所说的，adapter 之所以耦合度比较高，是因为它同时实现了数据集，视图的创建、绑定。所以解耦的第一个思路自然是把数据与试图分开。观察平常 Adapter 的使用，可以看到 Adapter 中关于数据，有几个比较重要的方法：







