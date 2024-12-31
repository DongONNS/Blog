---
title: MicroMeter笔记
date: 2020-04-21 22:00:00
tags: 
categories: Java
cover: /img/MicroMeter.png
---

<!--more-->



应用通过micrometer采集和暴露监控端点给prometheus，prometheus通过pull模式来采集监控时序数据信息。之后作为数据源提供给grafana进行展示。

关于MicroMeterde的注册表，MicroMeter里面的Meters是从MeterRegistry中创建并保存在其中的，而且每一个监控系统都有他们自己的MeterRgistry实现，在我们的MicroMeter里面也有自带的SimpleMeterRegistry，所以我们在项目中导入了MicroMeter的依赖以后就可以生成一部分的指标了，但是这部分指标要能够被收集到注册表上才能到我们的监控系统上；所以就由我们的复合注册表，将其他的注册表添加到复合注册表就能被多个监控系统监控；还有全局注册表，这个也是个复合注册表，Metrics.addRegistry(xxxxx);基于此我们就可以使用Metrics.counter这种方式生成指标了；

![实现流程](https://img2018.cnblogs.com/blog/285763/201812/285763-20181226164352509-830589043.png)

## 怎么定义一个Metric？
基于上述我们能得到一个结论就是一个指标的生成方法就是———注册表.指标类型（name,tag{key},tag{value},......），我们的Metris.指标类型（name,tag{key},tag{value},......）也相当于是全局注册表在干这件事情；

## 什么是meters？
我觉得Meter与Timer，Counter，Gauge，DistributionSummary，LongTaskTimer，FunctionCounter，FunctionTimer，和TimeGauge
的关系就像是集合与Set、Map、List的关系一样，就是一个总分关系；


## 怎么指定一些通用的标签？
可以在注册表级别上进行定义，定义的代码是：
registry.config().commonTags("stack", "prod", "region", "us-east-1")

这样监控系统上每个metrics上面就能找到定义的这些标签了。

## 什么是MeterFilter？
MeterFilter主要有三个功能：

	1.拒绝（或接受）meter被注册。

	2.转换Meter的ID（例如，更改名称，添加或删除标签，更改描述或基本单位）。

	3.配置某些meter类型的分布统计信息。

比如我们需要在注册表中忽略某个指标或者拒绝某个指标，下面即使通过编程的方式实现过滤信息：

registry.config()
.meterFilter(MeterFilter.ignoreTags("too.much.information"))
.meterFilter(MeterFilter.denyNameStartsWith("jvm"));

上面忽略了“too.much.information”这个指标，同时拒绝接受以jvm开头的指标，这个先知道有这些功能，具体的可以看翻译的[博客MeterFilter部分](https://blog.csdn.net/dongcheng_2015/article/details/103059035)

关于速率数据是怎么来的？
他的rate数据是来自于上一个时间间隔，例如下图数据
![rate](https://imgconvert.csdnimg.cn/aHR0cHM6Ly93eDQuc2luYWltZy5jbi9tdzY5MC8wMDc4NTdOWWx5MWc4djhncXp4YzFqMzFjbTBmcGp0ci5qcGc?x-oss-process=image/format,png)

## 关于Counter类型？
Counter类型报告了一个指标--计数。
Counter类型指标的注册方法有两种：


    Counter counter = registry.counter("counter");
    
    Counter counter = Counter.builder("counter")
				  			.baseUnit("beans")//可选
				 			.description("a description of       what this counter does") //  optional
    			 			 .tags("region", "test") // optional
    			 			 .register(registry);

## 关于Gauge类型

Gauge是获取**当前值**的处理器，Gauge的典型示例是运行状态下的**集合或映射的大小或线程数**,Gauge的特点就是返回即时的数据，该MeterRegistry接口包含用于构建Gauge以观察数值、方法、、集合和映射的方法。


    List<String> list = registry.gauge("listGauge", Collections.emptyList(), new ArrayList<>(), List::size); (1)

    List<String> list2 = registry.gaugeCollectionSize("listSize2", Tags.empty(), new ArrayList<>()); (2)

    Map<String, Integer> map = registry.gaugeMapSize("mapGauge", Tags.empty(), new HashMap<>());

Guage的构造方法：

    Gauge gauge = Gauge
	    .builder("gauge", myObj, myObj::gaugeValue)
	    .description("a description of what this gauge does") // optional
	    .tags("region", "test") // optional
	    .register(registry);


## Timer类型

Timer的构造方法：
    Timer timer = Timer
	    .builder("my.timer")
	    .description("a description of what this timer does") // optional
	    .tags("region", "test") // optional
	    .register(registry);

Timer相当于一个用于测量时间的DistributionSummary，所以要测量时间我们都直接使用Timer,而非DistributionSummary。