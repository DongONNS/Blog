---
title: 什么是fail-fast机制？
date: 2019-11-04 20:00:56
tags:
categories: java
---

<!--more-->

## 什么是快速失败机制？
>**fail-fast** 机制，即快速失败机制，是java集合(Collection)中的一种错误检测机制。当在**迭代集合的过程中该集合在结构上发生改变的时候，就有可能会发生fail-fast**，即抛出ConcurrentModificationException异常。fail-fast机制并不保证在不同步的修改下一定会抛出异常，它只是尽最大努力去抛出，所以这种机制一般仅用于检测bug。

快速失败机制经常出现在我们的集合操作中，快速失败机制是Iterator区别于Enumeration的一大特点。下面是一个fail-fast的示例。

![fail-fast](https://wx1.sinaimg.cn/mw690/007857NYly1g8ma643in5j30y20g9myj.jpg)

上面的list存储的是String类型的数据，然后将这个list类型的数据进行遍历时删除索引为3的元素值为3的数据值，但是list存储的是String类型所以会出发快速失败机制，日志中会产生ConcurrentModificationException异常，如下所示：

![异常截图](https://wx1.sinaimg.cn/mw690/007857NYly1g8mad25cqwj31ee07vgmi.jpg)