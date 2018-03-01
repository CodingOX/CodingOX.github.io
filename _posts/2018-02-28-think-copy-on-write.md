---
layout: post
title: 我对CopyOnWrite的思考
categories: [并发]
description: 我对CopyOnWrite的思考
keywords: CopyOnWrite,并发
---

# 我对CopyOnWrite的思考

> `CopyOnWrite` 后文中表述为 `COW`

`CopyOnWrite`容器即`写的时候复制一个新的容器进行写`：通俗的理解是当我们往一个容器添加元素的时候，不直接往`当前容器`添加，而是先将`当前容器`进行Copy，复制出一个`新的容器`，然后在`新的容器`里添加元素，添加完元素之后，再将原容器的引用指向新的容器。

## 为什么要这么做？

我们需要了解到一个异常叫做：`ConcurrentModificationException`。

> 该异常产生的原因时：在对容器遍历的时候对其进行操作。比如说`foreach`遍历`List`的时候往其中`add`元素。

了解到`ConcurrentModificationException`异常后，我们就可以结合`COW`进行思考，如果写操作的时候不复制一个容器，仍然是之前的容器，那么此时并发的读操作就是对`之前容器`进行的操作，一个容器在被读的时候，又被另外一个线程进行了写操作，会报出上述错误。

所以CopyOnWrite容器也是一种读写分离的思想，读和写不同的容器，不会发生`ConcurrentModificationException`异常

## COW带来的好处！

可以对`CopyOnWrite`容器进行并发的读，而不需要加锁，因为当前容器不会添加任何元素。

## 对比Concurrent容器

- 最大的优势就是`COW容器`在被写的时候，仍然是可以读的。而`Concurrent容器`在写的时候，不能读。
- 不足1：`COW容器`在写入的时候会进行内部容器的复制，所以内部实现上多了一份核心数据的拷贝赛所需的资源，可以理解为：`拿空间换时间`
- 不足2：`COW容器`仅仅保证了数据的最终一致性，`Concurrent容器`保证了数据随时的一致性。

## 适用场景

- 对数据在操作过程中的一致性要求不高
- 根据上述不足1进行分析可以得出：更适用于读大于写的场景。换言之`COW容器`中保存的数据应该是尽可能不变化的。