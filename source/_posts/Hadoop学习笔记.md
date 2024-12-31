---
title: Hadoop学习笔记
date: 2019-10-27 15:04:26
categories: hadoop
tags:
---

<!--more-->
## HDFS概述
**NameNode(nn)**:存储文件的元数据，如文件名、文件目录结构、文件属性（生成时间、副本数、文件权限），以及每个文件的块列表和和块所在的DataNode等。

**DataNode(dn)**:在本地文件系统存储文件数据块以及数据块的校验和。

**Secondary NameNode(2nn)**:用来监控HDFS的辅助后台程序，每隔一段时间获取HDFS元数据的快照。

yarn架构图

常见的正则表达式



## 为什么不能一直格式化NameNode,格式化NameNode需要注意什么？

集群在工作的时候他的datanode的集群id和namenode的集群id是一样的，这样才能保证集群的正常工作，如果你格式化了namenode的集群id这个时候你的datanode的集群id是没有改变的，他们两个不一致的话集群无法正常工作。

如果需要格式化NameNode的信息，那就需要datanode中的关于集群id的信息删除；

![格式化](https://wx2.sinaimg.cn/mw690/007857NYly1g8l91bg3iuj31h20oyqjl.jpg)


关于YARN

ResourceManager:集群资源的总管
NodeManager:单个节点资源的总管

## HDFS的优缺点
> 优点

>> 1.高容错性
>> 
>> 2.适合处理大数据
>> 
>> 3.可构建在廉价机器上

> 缺点
> >1.不适合低延时数据访问
> >
> >2.无法高效的对大量小文件进行存储
>>> 1.存储大量小文件的话，会占用NameNode的大量内存来存储文件目录和块信息，但NameNode的内存总是有限的
>
>>> 2.小文件的存储时间会超过读取时间，违反了DFSDE的设计目标
> >
> >3.不支持并发写入，文件的随机修改
>>> 一个文件只能有一个写，不允许多个线程同时写
>
>>> 仅支持数据的追加，不支持数据的随机修改