---
title: RecycleView Adapter 的解耦实现
author: Jiacheng
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

回到前面所说的，adapter 之所以耦合度比较高，是因为它同时实现了数据集，视图的创建、绑定。所以解耦的第一个思路自然是把数据与视图抽离出来。

于是，将 Adapter 中关于数据集的几个关键方法抽成一个接口：

``` java
public interface DataSource<Model> {

    int getDataType(int position);

    int getDataCount();

    Model getData(int position);
}
```

这个接口已经包含获取数据对应的Item 类型，Item 个数，以及指定 position 的数据的方法，已经能够给 Adapter 提供数据了。

Adapter 中关于视图的方法主要有两个：`onCreateViewHolder()` ， `onBindViewHolder()`，一个负责视图的创建，一个负责视图的绑定。把这个两个主要的方法单独抽象出来。

然而对于不同的Item类型可能会需要不同的ViewHolder，考虑到ViewHolder只是为了 Hold 住对应视图的ItemView，以及它的子View，所以创建一个通用的ViewHolder类来实现这一功能：

``` java 
@SuppressWarnings("WeakerAccess")
public class SuperViewHolder extends RecyclerView.ViewHolder {

    private SparseArray<View> viewHolder = new SparseArray<>();

    public SuperViewHolder(View itemView) {
        this(itemView, new int[0]);
    }

    public SuperViewHolder(View itemView, @IdRes int... ids) {
        super(itemView);
        holderChildViewByIds(ids);
    }

    public void holderChildViewByIds(@IdRes int... ids) {
        for (int id : ids) {
            viewHolder.put(id, itemView.findViewById(id));
        }
    }

    @SuppressWarnings("unchecked")
    public <T extends View> T get(@IdRes int id) {
        View view = viewHolder.get(id);
        if (view == null) {
            view = itemView.findViewById(id);
            viewHolder.put(id, view);
        }
        return (T) view;
    }
}
```

使用一个 SparseArray 来保存已经被find的子View，并使用只view的id作为key。可以在构造 SuperViewHolder 的时候传入需要 hold 的 view， 传入需要用到的子 View 的 Id 进行 hold，也可以在合适时机通过 `holderChildViewByIds()`，否则只有通过 `get(int id)` 的时候才会find子view。


