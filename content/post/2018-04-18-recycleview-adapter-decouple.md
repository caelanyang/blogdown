---
title: RecycleView Adapter 的解耦过程
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

前段时间由于 APP 业务上的原因，写了一个相对比较复杂的 RecycleView 的 Adapter，这样就导致 adapter 里面集中了大量的代码。在其中的 `onCreateViewHolder()`，`onBindViewHolder()`,以及 `getItemViewType` 这三个方法里面堆积了大量的 if-else，特别是 RecycleView 的 item 类型比较多的时候。这三个方法基本都要同时修改，添加逻辑分支。
