---
title: MicroMeter的基础概念
date: 2021-01-26 22:00:00
tags:
categories: Monitor System
---

## 表格的语法

| Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |

首行缩进两字符：&emsp;&emsp;

MicroMeter中指标的分类包括五种类型:timers, gauges, counters, 
distribution summaries, and long task timers

# 一、目的
&emsp;&emsp;MicroMeter是针对基于JVM的应用程序的Metrics标准检测库。它为最流行的监视系统提供了一个基于可视化客户端的简单外观，使您无需供应商锁定即可对基于JVM的应用程序代码进行可视化。它旨在在最大限度地提高指标工作可移植性的同时，几乎不增加指标收集活动的开销。

---

# 二、支持的监控系统
&emsp;&emsp;MicroMeter不是分布式跟踪系统或事件日志记录器。阿德里安·科尔（Adrian Cole）关于“ 可观测性3种方式”的演讲在强调这些不同类型的系统之间的差异方面做得很好。

&emsp;&emsp;MicroMeter包含一个带有可视化一起SPI的核心模块，一组包含各种监视系统的实现（每个系统称为注册表）的模块，以及一个测试套件。学习监视系统理解下面三个重要特征：

## 2.1 维度
&emsp;&emsp;系统是否支持通过标记键/值对丰富metrics名称（tags）。如果系统不是为维度性的，则它是分层的，这意味着它仅支持平面度量标准名称。将metrics指标发布到分层系统时，Micrometer会展平标签键/值对的集合并将其添加到名称中。
## 2.2 速率聚合（Rate aggregation）
&emsp;&emsp;在本文中，我们指的是在规定的时间间隔内集合一组样本。一些监视系统希望某些类型的离散样本（例如计数）在发布之前由应用程序转换为比率。有些人希望总是发送累积值。还有其他人对此没有意见。
## 2.3 发布
&emsp;&emsp;一些系统希望在闲暇时轮询应用程序以获取指标，而另一些系统则希望按固定间隔将指标推送给他们。

---

# 三、注册表（Registry）
&emsp;&emsp;Meter是用于收集有关应用程序的一组度量（我们分别称为metrics指标）的接口。MicroMeter里面的Meters是从MeterRegistry中创建并保存在其中的。每个支持的监视系统都有一个的实现的MeterRegistry。注册表的创建方式因每个监控系统的实现方式而不同。

&emsp;&emsp;MicroMeter包装带有SimpleMeterRegistry，可在内存中保存每个meter的最新值，并且不会将数据导出到任何地方。如果您还没有首选的监视系统，则可以使用简单的注册表开始使用指标：

    MeterRegistry registry = new SimpleMeterRegistry();

注：一个SimpleMeterRegistry是基于Spring的应用程序自动注入的。

## 3.1 复合注册表
&emsp;&emsp;MicroMeter提供了一个CompositeMeterRegistry可以添加多个注册表的功能，使我们可以同时将Metrics指标发布到多个监视系统。

    CompositeMeterRegistry composite = new CompositeMeterRegistry();
    
    Counter compositeCounter = composite.counter("counter");
    compositeCounter.increment(); (1)

    SimpleMeterRegistry simple = new SimpleMeterRegistry();
    composite.add(simple); (2)
    
    compositeCounter.increment(); (3)

1.组合中没有注册表之前，增量为NOOPd。此时计数器的计数仍将为0。

2.一个名为“ counter”的计数器被注册到简单注册表中。

3.简单注册表计数器以及组合中任何其他注册表的计数器都会增加。 

## 3.2 全局注册表
&emsp;&emsp;MicroMeter提供了一个静态全局注册表Metrics.globalRegistry和一组静态生成器，用于基于此注册表生成meters。globalRegistry是一个复合注册表。(我是不是可以理解成Metrics就是默认完成了那个Metrics.globalRegistry，然后我们就可以肆无忌惮的使用Mtrics.xxx了)

    class MyComponent {
    Counter featureCounter = Metrics.counter("feature", "region", "test"); (1)

    void feature() {
        featureCounter.increment();
    }

    void feature2(String type) {
        Metrics.counter("feature.2", "type", type).increment(); (2)
        }
    }

    class MyApplication {
    void start() {
        // wire your monitoring system to global static state
        Metrics.addRegistry(new SimpleMeterRegistry()); (3)
		}    
    }

&emsp;&emsp;1.只要能够实现（尤其是在可视化性能至关重要的地方），请将Meter实例存储在字段中，以避免在每次使用时查找其名称/标签。

&emsp;&emsp;2.当需要根据本地上下文确定标记时，您别无选择，只能在方法体内构造/查找Meter。查找成本只是单个哈希查找，因此对于大多数用途而言，这是可以接受的。

&emsp;&emsp;3.创建meters后添加注册表也是可以的，例如Metrics.counter(…​)。这些meters将被添加到每个注册表，因为它已绑定到全局组合。

## 3.4 Meters
&emsp;&emsp;MicroMeter包含的它支持Meter原语集包括：Timer，Counter，Gauge，DistributionSummary，LongTaskTimer，FunctionCounter，FunctionTimer，和TimeGauge。不同的meter类型得出不同数量的时间序列指标metrics。例如，虽然有一个单独的表示Gauge类型的指标，但是Timer类型的指标既得出时间事件的计数，也含有时间事件总的计数。

&emsp;&emsp;meter是通过其名称和维度进行唯一标识的。我们可以“维度”和“标签”是一个意思，而“MicroMeter”层面Tag仅仅是因为它更短。作为一般规则，应该可以使用该名称作为枢轴。维度允许时间切面以特定的命名指标进行深入分析并推断数据。这意味着，如果仅选择名称，则用户可以使用其他维度和显示值的原因进行追溯。

# 五 命名meters

&emsp;&emsp;MicroMeter采用命名约定，用“.”（点）字符分隔小写单词。不同的监视系统对命名约定有不同的建议，并且某些命名约定可能与一个系统兼容，而与另一个系统不兼容。监视系统的每个Micrometer实施都附带一个命名约定，该约定将小写点标记法名称转换为监视系统推荐的命名约定。此外，此命名约定实现还清除了监视系统不允许的Metrics标准名称和特殊字符标签。您可以使用以下方法在注册表中实现NamingConvention并设置注册表，从而覆盖注册表的默认命名约定：

    registry.config().namingConvention(myCustomNamingConvention);

&emsp;&emsp;有了适当的命名约定，在Micrometer中注册的以下timer计时器在各种监视系统中就可以很好的使用了：

    registry.timer("http.server.requests");

&emsp;&emsp;Prometheus- http_server_requests_duration_seconds

&emsp;&emsp;Atlas- httpServerRequests

&emsp;&emsp;Graphite- http.server.requests

&emsp;&emsp;InfluxDB- http_server_requests

&emsp;&emsp;通过遵守Micrometer的小写点标记约定，我们就可以确保跨监视系统的metrics名称具有最大的可移植性。

## 5.1 标签命名

>tip
>>命名标签时，建议使用与metrics名称相同的小写点标记。利用标签的这种一致的命名约定，可以更好地转换为相应监视系统的惯用命名方案。

假设我们正在尝试测量http请求的数量和数据库调用的数量。

推荐方法：

    registry.counter("database.calls", "db", "users")

    registry.counter("http.requests", "uri", "/api/users")

&emsp;&emsp;这个变量提供了足够的文本信息，因此，如果仅选择名称，则可以推断出该值，并且至少具有潜在的意义。例如，如果我们选择“database.calls”则可以看到所有数据库的调用总数。然后，我们可以分组或选择“db”以进一步细分或对呼叫对每个数据库的贡献进行比较分析。

不合适的方法

    registry.counter("calls",
    				"class", "database",
    				"db", "users");


    registry.counter("calls",
    				"class", "http",
    				"uri", "/api/users");
&emsp;&emsp;在这种方法中，如果选择，calls我们将获得一个值，该值是对数据库和API端点的调用次数的总和。如果没有进一步的维度向下钻取，该时间序列将无用。

## 5.2 通用标签
&emsp;&emsp;可以在注册表级别定义公用标签，并将其添加到报告给监视系统的每个度量标准中。通常用于在操作环境（例如主机，实例，区域，堆栈等）上进行维度深入分析。

    registry.config().commonTags("stack", "prod", "region", "us-east-1");
    registry.config().commonTags(Arrays.asList(Tag.of("stack", "prod"), Tag.of("region", "us-east-1"))); 
    // equivalently

&emsp;&emsp;调用commonTags追加其他通用标签。

>重要
>>如果我们在Spring环境中，需要通过添加MeterRegistryCustomizerBean来添加通用标签，以确保在自动配置meter绑定程序之前应用了通用标签。

## 5.3 标签值
&emsp;&emsp;标签值必须非空

>警告
>>注意来自用户提供的来源的标记值可能会破坏指标的基数。我们应该始终保持规范并绑定用户提供的输入。有时原因是非常细微的。考虑用于记录服务端点上的HTTP请求的URI标记。如果我们不将404约束为NOT_FOUND之类的值，则metrics的维度将随着找不到的每个资源而增长。

---

# 6.Meter过滤器
&emsp;&emsp;每个注册表都可以配置meter过滤器，使我们可以更好地控制如何注册仪表，何时注册仪表以及发出何种统计信息。Meter过滤器具有三个基本功能：

&emsp;&emsp;1.拒绝（或接受）meter被注册。

&emsp;&emsp;2.转换Meter的ID（例如，更改名称，添加或删除标签，更改描述或基本单位）。

&emsp;&emsp;3.配置某些meter类型的分布统计信息。

&emsp;&emsp;MeterFilter的实现以编程方式添加到注册表中：

    registry.config()
    .meterFilter(MeterFilter.ignoreTags("too.much.information"))
    .meterFilter(MeterFilter.denyNameStartsWith("jvm"));

&emsp;&emsp;按顺序应用Meter过滤器，并将变换或配置meter的结果链接在一起。

## 6.1 拒绝/接受meters

&emsp;&emsp;接受/拒绝过滤器的详细形式为：

    new MeterFilter() {
	    @Override
	    public MeterFilterReply accept(Meter.Id id) {
	       if(id.getName().contains("test")) {
	          return MeterFilterReply.DENY;
	       }
	       return MeterFilterReply.NEUTRAL;
	    	}
  	  }
&emsp;&emsp;MeterFilterReply具有三种可能的状态：

- DENY-请勿注册该仪表。当我们尝试注册一个与另一个注册表冲突的meter,过滤器返回DENY注册表将返回meter的NOOP版本（例如NoopCounter，NoopTimer）。我们的代码可以继续与NOOP计量器进行交互，但是记录在其中的任何内容都将以最小的开销立即被丢弃。

- NEUTRAL-如果没有其他meter过滤器返回DENY，则meter的注册将照常进行。

- ACCEPT-如果过滤器返回ACCEPT，meter将立即注册，而不会询问任何其他过滤器的接受方法。
	
### 6.1.1 便利的方法

### 6.1.2 链接 拒绝/接受meters

## 6.2 转换metrics

&emsp;&emsp;转换过滤器如下所示：

    new MeterFilter() {
	    @Override
	    public Meter.Id map(Meter.Id id) {
	       if(id.getName().startsWith("test")) {
	          return id.withName("extra." + id.getName()).withTag("extra.tag", "value");
	       }
	       return id;
	    }
    }

&emsp;&emsp;该过滤器有条件地向meters添加以名称“ test”开头的名称前缀和附加标签。

&emsp;&emsp;MeterFilter 为许多常见的转换案例提供了便利构建器：

&emsp;&emsp;commonTags(Iterable<Tag>)-向所有指标添加一组标签。强烈建议为应用名称，主机，区域等添加通用标签。

&emsp;&emsp;ignoreTags(String…​)-从每个仪表上舍弃匹配的标签键。当标签可证明变得过高的基数并开始给监视系统施加压力或开销增大，使得我们不能快速更改所有检测点时，此功能特别有用。

&emsp;&emsp;replaceTagValues(String tagKey, Function<String, String> replacement, String…​ exceptions)-根据为所有匹配的标记键提供的映射替换标记值。通过将标签值的某些部分映射到其他内容，可以用来减少标签的总基数。

&emsp;&emsp;renameTag(String meterNamePrefix, String fromTagKey, String toTagKey) -重命名以给定前缀开头的每个指标的标签键。


## 6.3 配置分布统计

&emsp;&emsp;Timer和DistributionSummary除了可以通过过滤器配置的计数、总数和最大值的基本信息外，还包含一组可选的分布统计信息。这些分布统计信息包括预先计算的百分位数，SLA和直方图。

    new MeterFilter() {
	    @Override
	    public DistributionStatisticConfig configure(Meter.Id id, DistributionStatisticConfig config) {
	        if (id.getName().startsWith(prefix)) {
	            return DistributionStatisticConfig.builder()
	                    .publishPercentiles(0.9, 0.95)
	                    .build()
	                    .merge(config);
	        }
	        return config;
	    }
    };

&emsp;&emsp;通常，我们应该仅对我们希望配置的切片创建一个新的DistributionStatisticConfig并且将它与输入的配置合并。这样，我们就可以下拉注册表提供的分发统计信息的默认值，并将多个过滤器链接在一起，每个过滤器都可以配置分发统计信息的一部分（例如，我们可能希望所有http请求都具有100ms的SLA，但在一些关键请求上只需要百分位端点）。

&emsp;&emsp;MeterFilter 为以下人员提供便利构建器：

&emsp;&emsp;maxExpected(Duration/long) -控制从timer或summary中得到的百分比直方图桶的上限。

&emsp;&emsp;minExpected(Duration/long) -控制从timer或summary中得到的百分比直方图桶的下限。

&emsp;&emsp;Spring Boot提供了基于属性的过滤器，用于通过名称前缀配置SLA，百分比和百分比直方图。

# 7.速率聚合
&emsp;&emsp;Micrometer知道特定的监视系统希望在metrics发布之前还是在服务器端作为查询的一部分临时在客户端进行速率聚合。它根据监视系统期望的样式累积metrics。

&emsp;&emsp;并非所有测量指标都进行速率聚合进行报告或展示。例如，gauge和long task timer的任务就不是速率聚合。 

&emsp;&emsp;执行服务器端速率聚合的监视系统希望在每个发布间隔报告绝对值。例如，自从应用程序的开始运行，在每个发布间隔所有counter增量的绝对值都会发布。

&emsp;&emsp;假设我们有一个略微偏向的随机游走，它选择每10ms增加一次counter计数器。如果我们在监控系统，比如Prometheus这样的系统中查看原始计数器值，则会看到逐步单调递增的函数（步长是Prometheus轮询或抓取数据的间隔）。

![rate1](https://wx4.sinaimg.cn/mw690/007857NYly1g8v8j33vttj30i708zmxx.jpg)

&emsp;&emsp;在某个时间窗口内没有速率聚合的counter很少是有用的，因为表示的是计数器递增的速度和服务寿命的函数。在上面的示例中，服务重新启动时，counter降回零。新实例（例如在生产部署中）投入使用后，速率聚合图像将返回到约55的值。

&emsp;&emsp;如果我们已实现零停机时间部署（例如通过红黑部署），则应该能够在速率汇总图上轻松设置最低警报阈值，而无需重新启动服务导致计数器值下降。

>重要
>>对于大多数生产目的，无论是警报，自动数据分析等，都是基于速率汇总数据进行自动化的。

## 7.2 客户端
另一类监视系统：

&emsp;&emsp;1.期望速率汇总数据。考虑到对于大多数生产目的的关键需求指标，我们应该基于速率而不是绝对值来做出决策，这样的系统将得益于无需进行太多数学运算即可满足查询的需求。

&emsp;&emsp;2.几乎没有数学运算或没有数学运算，即可使我们可以通过查询对数据进行速率聚合。对于这些系统，发布预先聚合的数据是构建有意义的表示的唯一方法。

&emsp;&emsp;MicroMeter通过累积当前发布间隔数据的步长值来有效维护速率数据。当对步长值进行轮询时（例如，在发布时），如果步长值检测到当前间隔已过去，则它将当前数据移至“先前”状态。该先前状态是报告的，直到下次当前数据覆盖它为止。以下是当前和先前状态以及轮询之间相互作用的说明：

&emsp;&emsp;poll函数返回的值始终是每秒的速率*间隔。如果上面说明的步长值表示counter的值，则可以说计数器在第一个间隔中看到了“每秒0.3增量”，这可以在第二个间隔中的任何时间报告给后端。

&emsp;&emsp;MicroMeterde的timer至少跟踪一个计数和总时间，作为单独的测量。假设我们以10秒的间隔配置发布，并且看到20个请求，每个请求花费100毫秒。然后在第一个间隔中：

    1.count = 10秒*（20个请求/ 10秒）= 20个请求

    2.totalTime = 10秒*（20 * 100毫秒/ 10秒）= 2秒

&emsp;&emsp;该count统计信息很有意义，可以单独使用--它是吞吐量的度量。totalTime表示间隔中所有请求的总延迟。另外：

    totalTime / count = 2秒/ 20个请求= 0.1秒/请求= 100 ms /请求

&emsp;&emsp;这是平均延迟的有用度量。当将相同的思想应用于分布汇总totalAmount并count从中得出时，该度量称为分布平均值。平均延迟只是按时间（timer）度量的分发summary的分发平均值。犹如Atlas一些监测系统计算从这些统计分布平均提供设施和MicroMeter将发布totalTime和count作为单独统计。其他如Datadog则没有内置此类操作，Micrometer将计算客户端的平均分配并将其发送出去。
![rate分析](https://wx4.sinaimg.cn/mw690/007857NYly1g8v8gqzxc1j31cm0fpjtr.jpg)
&emsp;&emsp;发布时间间隔的速率足以推断出在任何时间窗口内大于或等于发布时间间隔的速率。在我们的示例中，如果服务在给定的分钟内继续接收20个请求，在每10秒间每个请花费100ms，那么我们可以说：

&emsp;&emsp;1.MicroMeter每10秒报告一次total--“ 20个请求” 。监视系统简单地将这六个10秒的间隔相加即可得出每分钟120个请求的结论。请注意，是进行进行求和操作的是监视系统，而不是MicroMeter。

&emsp;&emsp;2.MicroMeter每隔10 秒报告一次totalTime--“ 2秒” 。监视系统可以对一分钟内的所有总时间统计信息求和，以在分钟间隔内产生总时间的“ 12秒”。然后，平均延迟就如我们期望的那样：12秒/ 120个请求= 100 ms /请求。

---

# 8.Counters
Couners报告一个指标，计数。该Counter接口允许我们以固定数量递增，该数量必须为正。

>tip
>>永远不要记数一些我们可以使用Timer或用DistributionSummary记录的东西！Timer和DistributionSummary会发布除了其他的测量事件之外，还有事件的计数。
>

&emsp;&emsp;建立图表并根据Counter发出警报时，通常我们应该最感兴趣的是测量在给定时间间隔内某些事件的速率。考虑一个简单的队列，Counter可用于衡量事物的插入和移除速度。

&emsp;&emsp;人们首先想到的是可视化绝对数值，而不是速率，但绝对数量通常是既是一些事务的使用速率的函数也是可视化条件下应用程序实例的使用寿命。在某些时间间隔内构建仪表板和Counter速率警报会忽略应用程序的寿命，让我们在应用程序启动后很长时间才看到异常行为。

>注意
>>在进入使用计数器之前，请务必通读Timer部分，因为Timer会记录计时事件的计数，因为Counter记录了时间事件的计数作为度量的一部分。对于我们打算计时的那些代码，我们无需单独添加计数器。

&emsp;&emsp;以下代码模拟了一个真实的Counter，该Counter的rate在较短的时间范围内显示出一些扰动。


    Normal rand = ...; // a random generator


    MeterRegistry registry = ...

    Counter counter = registry.counter("counter"); (1)


    Flux.interval(Duration.ofMillis(10))
        .doOnEach(d -> {
            if (rand.nextDouble() + 0.1 > 0) { (2)
                counter.increment(); (3)
            }
        })
        .blockLast();


&emsp;&emsp;1.可以使用名称和（可选）一组标签从注册表本身创建大多数Counter。

&emsp;&emsp;2.略微偏向的随机游走。

&emsp;&emsp;3.这就是我们与Conuter交互的方式。我们也可以用counter.increment(n)在单个操作中调用增加1以上的值。

&emsp;&emsp;Counter接口本身还有一个流利的计数器构建器，可以访问不太常用的选项，例如基本单位和说明。我们可以通过调用将计数器注册为构造计数器的最后一步register。

    Counter counter = Counter
	    .builder("counter")
	    .baseUnit("beans") // optional
	    .description("a description of what this counter does") // optional
	    .tags("region", "test") // optional
	    .register(registry);

## 8.1 功能--追踪Counter

&emsp;&emsp;MicroMeter还提供了一种更不常用的计数器模式，该模式跟踪单调递增的函数（函数保持不变或随时间增加，但从不减少）。一些监视系统（例如Prometheus）将Counter的累积值推送到后端，但是其他系统则发布Counter在推送间隔内递增的速率。通过采用这种模式，我们可以让监控系统的Micrometer实施选择是否对Counter进行归一化评估，并且计数器可以在不同类型的监控系统之间保持可移植性。


    Cache cache = ...; // suppose we have a Guava cache with stats recording on

    registry.more().counter("evictions", tags, cache, c -> c.stats().evictionCount()); (1)

&emsp;&emsp;1.evictionCount() 是一个单调递增的函数，从生命周期的开始就随着每次缓存逐出而递增。

&emsp;&emsp;功能--跟踪Counter,与监视系统的速率归一化功能（无论是查询语言的人工产物还是数据被推送到系统的方式）配合使用，在功能的累积值之上增加了一层丰富性。可以推断值增加的速率，这样的速率是否在一个可接受的范围，随着时间的推移是在增加或降低。

>警告
>>MicroMeter不能为您保证函数的单调性。通过使用此签名，我们可以根据对定义的了解来断言其单调性。
>
&emsp;&emsp;FunctionCounter接口本身也具有用于功能计数器的流利的构建器，可以访问不太常用的选项，例如基本单元和描述。我们可以通过调用将计数器注册为构造计数器的最后一步register(MeterRegistry)。

    MyCounterState state = ...;

    FunctionCounter counter = FunctionCounter
	    .builder("counter", state, state -> state.count())
	    .baseUnit("beans") // optional
	    .description("a description of what this counter does") // optional
	    .tags("region", "test") // optional
	    .register(registry);

# 9.Gauge

&emsp;&emsp;Gauge是获取当前值的处理器。Gauge的典型示例是运行状态下的**集合或映射的大小或线程数**。

>tip
>>Gauge对于监视具有自然上限的事物很有用。不建议使用Gauge来监视诸如请求计数之类的事情，因为它们会在应用程序实例生命周期内不受限制地增长。

>tip
>>能用Counter计量的东西千万不要用Gauge！

&emsp;&emsp;MicroMeter的立场是应该对Gauge进行采样而不进行设置，因此没有关于样品之间可能发生的情况的信息。毕竟，在将Gauge值报告给Metrics后端后，Gauge上设置的任何中间值都会丢失，因此首先设置这些中间值似乎没有什么价值。

&emsp;&emsp;如果有帮助，可以将其Gauge视为“heisen-gauge”-仅在观察到时才会改变的仪表。开箱即用提供的所有其他Meter类型都会在向数据后端发送数据时累积中间计数。

&emsp;&emsp;该MeterRegistry接口包含用于构建Gauge以观察数值、方法、、集合和映射的方法。

    List<String> list = registry.gauge("listGauge", Collections.emptyList(), new ArrayList<>(), List::size); (1)
    List<String> list2 = registry.gaugeCollectionSize("listSize2", Tags.empty(), new ArrayList<>()); (2)
    Map<String, Integer> map = registry.gaugeMapSize("mapGauge", Tags.empty(), new HashMap<>());

&emsp;&emsp;1.gauge的一种较常见的形式是监视某些非数字对象。最后一个参数建立了一个函数，该函数用于在观察gauge时确定gauge的值。

&emsp;&emsp;2.对于(1)的一种更方便的形式，用于您只想监视集合大小的情况。

&emsp;&emsp;创建gauge的所有不同形式都仅保持对要观察的对象的弱引用，以免阻止对象的垃圾收集。

## 9.1 手动增减仪表

&emsp;&emsp;gauge可制成可以跟踪任何java.lang.Number亚型，如java.util.concurrent.atomic中发现的AtomicInteger和AtomicLong，和Guava的AtomicDouble类似的类型。

    AtomicInteger n = registry.gauge("numberGauge", new AtomicInteger(0));
    n.set(1);
    n.set(2);

&emsp;&emsp;请注意，这种形式与其他meter类型不同，Gauge在创建一个meter时，我们不会得到引用，而是要观察到的东西。这是因为存在"heisen-gauge"原理，gauge一经创建便是自给自足的，因此我们无需与之交互。这使MicroMeter可以只向我们退还已检测的对象，从而可以快速创建一个可观察对象并围绕该对象设置metrics指标。

&emsp;&emsp;此模式应比DoubleFunction表格少见。请记住，频繁设置观察值会Number导致许多中间值无法发布。仅在发布时将gauge的瞬时值发送到监视系统。

>警告
>>尝试使用原始数字或其java.lang对象形式之一构造gauge总是不正确的。这些数字是不可变的，因此gauge无法更改。尝试用不同的编号“重新注册”量规是行不通的，因为注册表仅对名称和标签组成的唯一组合维护一个仪表。

## 9.2 量规的构建器

&emsp;&emsp;该接口包含一个流利的gauge构建器：

    Gauge gauge = Gauge
	    .builder("gauge", myObj, myObj::gaugeValue)
	    .description("a description of what this gauge does") // optional
	    .tags("region", "test") // optional
	    .register(registry);

&emsp;&emsp;通常，返回的Gauge实例仅在测试中有用，因为gauge已设置为在注册后自动跟踪数值。

## 9.3 我的仪表为什么报告NaN或消失？

&emsp;&emsp;MicroMeter对于创建强引用对象非常谨慎，避免那些对象无法进行垃圾收集，一旦被测量的对象被取消引用并被垃圾回收，Micrometer将开始报告测量值的NaN或不报告，具体取决于注册表的实现方式。

&emsp;&emsp;如果看到gauge报告了几分钟，然后消失或报告了NaN，则几乎可以肯定地表明被测量的基础对象已被垃圾回收。

---

# 10 计时器

&emsp;&emsp;计时器用于测量短时延以及此类事件的频率。所有实现的Timer将按照时间序列报告至少总时间和事件计数。尽管可以将Timers用于其他用例，但请注意不支持负值，并且记录更长的持续时间可能会导致总时间溢出（以Long.MAX_VALUE纳秒计）（292.3年）。

&emsp;&emsp;例如，考虑一个图表，它显示了对典型Web服务器的请求延迟。可以期望服务器快速响应许多请求，因此计时器将每秒更新多次。

&emsp;&emsp;Counter的适当基本单位因指标后端的不同而有所不同。MicroMeter对此毫无疑问，但是由于存在混淆的可能性，与Timers 相互作用时需要TimeUnit。Micrometer知道每种实现的首选项，并根据实现在适当的基本单元中发布我们的时间安排。


    public interface Timer extends Meter {
	    ...
	    void record(long amount, TimeUnit unit);
	    void record(Duration duration);
	    double totalTime(TimeUnit unit);
    }

&emsp;&emsp;该接口包含一个流畅的Timer构建器：


    Timer timer = Timer
    .builder("my.timer")
    .description("a description of what this timer does") // optional
    .tags("region", "test") // optional
    .register(registry);

## 10.1 记录代码块

&emsp;&emsp;该Timer接口公开了一些方便的重载，用于内联时间记录，例如：

    timer.record(() -> dontCareAboutReturnValue());
    timer.recordCallable(() -> returnValue());

    Runnable r = timer.wrap(() -> dontCareAboutReturnValue()); (1)

    Callable c = timer.wrap(() -> returnValue());

1.包装Runnable或Callable返回其可视化版本以供以后使用。

>注意
>>Timer实际上只是一个专门的分布式summary，它知道如何将持续时间缩放到每个监视系统的基本时间单位，并且具有自动确定的基本单位。在每种情况下，如果要测量时间，都应使用Timer而不是DistributionSummary。


## 10.2 将开始状态存储在Timer.Sample

&emsp;&emsp;您也可以将启动状态存储在示例实例中，以后可以将其停止。该示例记录基于注册表时钟的开始时间。开始采样后，执行要计时的代码，并通过调用stop(Timer)采样完成操作。

    Timer.Sample sample = Timer.start(registry);

    // do stuff

    Response response = ...

    sample.stop(registry.timer("my.timer", "response", response.status()));

&emsp;&emsp;请注意，在停止采样之前，如何确定采样所用的计时器。这使我们能够根据我们正在计时的操作的结束状态动态地确定某些标签。

## 10.3 @Timed注解

&emsp;&emsp;micrometer-core模块包含一个@Timed注释，框架可以使用该注释向特定类型的方法（例如为Web请求端点提供服务的方法）或通常为所有方法添加计时支持。

>警告
>>Micrometer的Spring Boot配置无法@Timed在任意方法上识别。

&emsp;&emsp;还包含一个可孵化的AspectJ方面micrometer-core，在应用程序中，我们可以通过编译/加载AspectJ编织，或通过框架工具比如Spring Aop等其他方式解释AspectJ切面和代理目标方法。如下是一个示例Spring AOP配置：

    @Configuration
    public class TimedConfiguration {
    @Bean
    public TimedAspect timedAspect(MeterRegistry registry) {
      return new TimedAspect(registry);
       }
    }

&emsp;&emsp;在AspectJ代理实例中，应用程序TimedAspect的@Timed用于任何任意方法。

## 10.4 功能--跟踪timers

&emsp;&emsp;MicroMeter还提供了一种不常用的计时器模式，该模式可跟踪两个单调递增的函数（一个函数随时间保持不变或增加，但从不减小）：计数函数和总时间函数。某些监视系统（例如Prometheus）将计数器的累积值（在这种情况下适用于计数和总时间函数）推送到后端，但其他系统则发布计数器在推送间隔内递增的速率。通过采用这种模式，我们可以让监控系统的Micrometer实现选择是否对timer进行速率聚合，并且计时器可以在不同类型的监控系统之间保持可移植性。

    IMap<?, ?> cache = ...; // suppose we have a Hazelcast cache
    registry.more().timer("cache.gets.latency", Tags.of("name", cache.getName()), cache,
	    c -> c.getLocalMapStats().getGetOperationCount(), (1)
	    c -> c.getLocalMapStats().getTotalGetLatency(),
	    TimeUnit.NANOSECONDS (2)
    );

&emsp;&emsp;1.getGetOperationCount() 是一个单调递增的函数，它从生命周期的开始就随着每个缓存的获取而递增。

&emsp;&emsp;2.这表示由getTotalGetLatency()表示的时间单位。每个注册表实现均指定其预期的基本时间单位是什么，并且报告的总时间将缩放为该值。

&emsp;&emsp;功能--跟踪timer与监视系统的速率归一功能（无论是查询语言的人工产物还是数据被推送到系统的方式）配合使用，在功能的累积值之上增加了一层丰富性，我们可以推断吞吐量和延迟的速率，该速率是否在可接受的范围内，随时间增加还是减少等。

>警告
>>MicroMeter不能为我们保证count和total time功能的单调性。通过使用此签名，您可以根据对定义的了解来断言它们的单调性。

&emsp;&emsp;FunctionTimer接口本身还有一个流利的FunctionTimer构建器，可用于访问不常用的选项，例如基本单元和描述。您可以通过调用register(MeterRegistry)为构造计时器的最后一步)。


    IMap<?, ?> cache = ...


    FunctionTimer.builder("cache.gets.latency", cache,
	        c -> c.getLocalMapStats().getGetOperationCount(),
	        c -> c.getLocalMapStats().getTotalGetLatency(),
	        TimeUnit.NANOSECONDS)
	    .tags("name", cache.getName())
	    .description("Cache gets")
	    .register(registry);

## 10.5 暂停检测

&emsp;&emsp;MicroMeter使用LatencyUtils包来补偿协同遗漏—系统和VM暂停导致的额外延迟会延迟您的延迟统计信息。百分位数和SLA计数之类的分布统计信息受暂停检测器实现的影响，该实现会在此处和此处添加额外的延迟以补偿暂停。

&emsp;&emsp;MicroMeter支持两种暂停检测器实施方式：基于时钟漂移的检测器和无操作检测器。在MicroMeter的1.0.10/1.1.4/1.2.0版本之前，默认情况下配置了时钟漂移检测器以报告尽可能准确的度量标准，而无需进行进一步配置。从1.0.10/1.1.4/1.2.0版本开始，默认情况下配置无操作检测器，但可以如下配置时钟漂移检测器。

&emsp;&emsp;基于时钟漂移的检测器具有可配置的睡眠间隔和暂停阈值。CPU消耗与sleepInterval、暂停检测精度成反比。这两个值的100ms是合理的默认值，以提供对长时间停顿事件的适当检测，同时消耗可忽略的CPU时间。

&emsp;&emsp;我们可以使用以下方法自定义暂停检测器：

    registry.config().pauseDetector(new ClockDriftPauseDetector(sleepInterval, pauseThreshold));

    registry.config().pauseDetector(new NoPauseDetector());

&emsp;&emsp;将来，我们可能会提供进一步的检测器实现。例如，在某些情况下，可以从GC日志中推断出一些暂停，而不需要恒定的CPU负载，尽管负载很小。将来的JDK也可能会提供对暂停事件的直接访问。

## 10.6 内存占用估算
&emsp;&emsp;Timer是最消耗内存的meter，其总占用空间可能会因我们选择的选项而有很大差异。下表是基于各种功能使用情况的内存消耗表。这些数字假定没有标签，并且环形缓冲区的长度为3。添加标签当然会增加总数，增加缓冲区的长度也会有所增加。总存储量也可能会有所不同，具体取决于注册表的实现。

- R =环形缓冲区的长度。在所有示例中，我们假定默认值为3。R用设置Timer.Builder#distributionStatisticBufferLength。

- B =总直方图桶。可以是SLA边界或百分比直方图桶。默认情况下，计时器被限制为最小期望值1ms和最大期望值30秒，如果适用，则为百分比直方图产生66个桶。

- I =暂停补偿的间隔估算器。1.7 kb


- M =最大衰减时间。104字节


- Fb =固定边界直方图。30b * B * R

- Pp =百分位数精度。默认情况下为1。通常在[0，3]范围内。Pp用设置Timer.Builder#percentilePrecision。

- Hdr（Pp）=高动态范围直方图。

若 Pp = 0：1.9kb * R + 0.8kb

若 Pp = 1：3.8kb * R + 1.1kb

若 Pp = 2：18.2kb * R + 4.7kb

若 Pp = 3：66kb * R + 33kb

| 暂停检测       | 客户端百分位数 | Cool  |直方图和/或SLA| 示例|
|:-------------:|:-------------:|:-----:|:-----------:|:-----:|
| YES           | NO            | NO    |I+M          |~1.8kb|
| YES           | NO            |   YES |I+M+Fb       |对于默认的百分比直方图，〜7.7kb|
| YES           | YES           |   YES |I+M+Hdr(Pp)  |要添加默认值的0.95％，否则为〜14.3kb|
| NO            | NO            |   NO  |M            |〜0.1kb|
|NO             | NO            |   YES |M+Fb         |对于默认的百分比直方图，〜6kb|
| NO            | YES           |   YES |M+Hdr(Pp)    |要添加默认值的0.95％，否则为〜12.6kb|


>注意
>>这些估计是基于Micrometer 1.0.3中所做的改进，并且假定至少使用该版本。

>注意
>>特别是对于Prometheus ，无论您如何尝试通过进行配置Timer.Builder，R 始终等于1 。这对于


&emsp;&emsp;Prometheus来说是特殊情况，因为它期望永不翻转的累积直方图数据。


# 11 分布式 summaries

&emsp;&emsp;分布式summaries用于跟踪事件的分布。它在结构上类似于计时器，但记录的值不代表时间单位。例如，分布式summary可用于衡量命中服务器的请求的有效负载大小。

&emsp;&emsp;要创建一个分布式summary：

    DistributionSummary summary = registry.summary("response.size");

&emsp;&emsp;该接口包含一个流利的分布式summary构建器：

    DistributionSummary summary = DistributionSummary
	    .builder("response.size")
	    .description("a description of what this summary does") // optional
	    .baseUnit("bytes") // optional (1)
	    .tags("region", "test") // optional
	    .scale(100) // optional (2)
	    .register(registry);

&emsp;&emsp;1.添加基本​​单元以获得最大的可移植性-基本单元是某些监视系统的命名约定的一部分。如果忘记的话，将其保留下来并违反命名约定不会有任何不利影响。

&emsp;&emsp;2.可选的，我们可以提供一个比例因子，每个记录的样本在记录时将乘以该比例。

## 11.1 标度和直方图
&emsp;&emsp;MicroMeter的预选百分比直方图桶都是从1到最大long的整数。目前minimumExpectedValue和maximumExpectedValue用来控制桶的基数。如果我们尝试检测到您的最小/最大值产生较小的范围并将预选的存储桶域缩放到summary的范围，则我们没有其他方法可以控制桶的基数。

&emsp;&emsp;相反，如果summary的域受到更多限制，则按固定因子缩放摘要的范围。到目前为止，我们所听到的用例是域为[0,1]的比率的汇总。然后：

    DistributionSummary.builder("my.ratio").scale(100).register(registry)

&emsp;&emsp;这样，比率最终会在[0,100]的范围内，我们可以将其设置maximumExpectedValue为100。如果您关心特定的比率，则将其与自定义SLA边界配对：

    DistributionSummary.builder("my.ratio")
       .scale(100)
       .sla(70, 80, 90)
       .register(registry)

## 11.2 内存占用估算

&emsp;&emsp;分布式summary的总内存占用量可能会变化很大，具体取决于我们选择的选项。下表是基于各种功能使用情况的内存消耗表。这些数字假定没有标签，并且环形缓冲区的长度为3。添加标签当然会增加总数，增加缓冲区的长度也会有所增加。总存储量也可能会有所不同，具体取决于注册表的实现。

- R =环形缓冲区的长度。在所有示例中，我们假定默认值为3。R用设置DistributionSummary.Builder#distributionStatisticBufferLength。

- B =总直方图桶。可以是SLA边界或百分比直方图桶。默认情况下，摘要没有最小和最大期望值，因此请装运所有276个预定的直方图桶。当您打算运送百分位直方图时，应始终使用minimumExpectedValue和来固定分布摘要maximumExpectedValue。

- M =最大衰减时间。104字节

- Fb =固定边界直方图。30b * B * R

- Pp =百分位数精度。默认情况下为1。通常在[0，3]范围内。Pp用设置DistributionSummary.Builder#percentilePrecision。

- Hdr（Pp）=高动态范围直方图。

当Pp = 0时：1.9kb * R + 0.8kb

当Pp = 1时：3.8kb * R + 1.1kb

当Pp = 2时：18.2kb * R + 4.7kb

当Pp = 3时：66kb * R + 33kb

| 客户端        | 百分位数直方图和/或SLA |  公式  |  示例  |
|:------------:|:---------------------:| -----:|:------:|
| NO           | NO                    | M     |〜0.1kb        |
| NO           | YES                   | M+Fb  |对于固定在66个桶中的百分位直方图，〜6kb|
| YES          | YES                   | M+Hdr(Pp) | 要添加默认值的0.95％，否则为〜12.6kb       |

>注意
>>这些估计是基于Micrometer1.0.3中所做的改进，并且假定至少使用该版本。
>
>注意
>>特别是对于Prometheus ，无论您如何尝试通过DistributionSummary.Builder进行配置，R 始终等于1。这对于Prometheus来说是特殊情况，因为它期望的是永不翻转的累积直方图数据。

---

# 12.长任务timers

&emsp;&emsp;长任务timers是一种特殊的timer，可让我们在正在测量的事件仍在运行时测量时间。在任务完成之前，计时器不会持续记录时间。

&emsp;&emsp;现在考虑一个后台过程来刷新数据存储中的元数据。例如，Edda缓存AWS资源，例如实例、卷、自动扩展组等。通常，所有数据都可以在几分钟内刷新。如果AWS服务出现问题，则可能需要更长的时间。长任务timer可用于跟踪刷新元数据的总时间。

&emsp;&emsp;例如，在Spring应用程序中，通常使用来实现如此长时间运行的进程@Scheduled。MicroMeter提供了特殊的@Timed注释，用于使用长任务timer来可视化这些过程。

    @Timed(value = "aws.scrape", longTask = true)
    @Scheduled(fixedDelay = 360000)
    void scrapeResources() {
    // find instances, volumes, auto-scaling groups, etc...
    }

&emsp;&emsp;@Timed是否生效取决于应用程序框架。如果所选择的框架不支持它，我们仍然可以使用长任务计时器：


    LongTaskTimer scrapeTimer = registry.more().longTaskTimer("scrape");

    void scrapeResources() {
	    scrapeTimer.record(() => {
	        // find instances, volumes, auto-scaling groups, etc...
	    });
    }

&emsp;&emsp;如果我们想在此过程超过阈值时发出警报，则需要longtasktimer，超过阈值后，我们将在第一个报告间隔收到该警报。如果使用常规timer，则直到该过程完成后的第一个报告间隔（超过一个小时），我们才会收到警报！

&emsp;&emsp;该接口包含用于Longtasktimer的流利的构建器：

    LongTaskTimer longTaskTimer = LongTaskTimer
	    .builder("long.task.timer")
	    .description("a description of what this timer does") // optional
	    .tags("region", "test") // optional
	    .register(registry);

---

# 13 直方图和百分位数

&emsp;&emsp;Timer和分布式summary支持收集数据以观察其百分比分布。查看百分位数的主要方法有两种：

&emsp;&emsp;1.百分比直方图 -MicroMeter将值累加为基础直方图，并将预定的一组buckets集合发送给监视系统。监视系统的查询语言负责计算此直方图的百分位数。目前，只有Prometheus，Atlas，和Wavefront支持直方图基于百分比近似，通过histogram_quantile，:percentile和hs()分别。如果定位到Prometheus，Atlas或Wavefront，则最好使用此方法，因为我们可以汇总各个维度上的直方图（通过简单地将一组维度中各个值的值相加）并从该直方图中得出可凝集的百分位数。

&emsp;&emsp;2.客户端百分位数 -MicroMeter为每个仪表ID（名称和标签集）计算百分位数近似值，并将百分位数值发送到监视系统。这不能像百分位数直方图那样灵活，因为不可能汇总标签之间的百分位数近似值。但是，它为不支持基于直方图的服务器端百分比计算的监视系统提供了一定百分比的洞察力。

&emsp;&emsp;这是一个使用直方图构建计时器的示例：

    Timer.builder("my.timer")

       .publishPercentiles(0.5, 0.95) // median and 95th percentile

       .publishPercentileHistogram()

       .sla(Duration.ofMillis(100))

       .minimumExpectedValue(Duration.ofMillis(1))

       .maximumExpectedValue(Duration.ofSeconds(10))

&emsp;&emsp;1.publishPercentiles-用于发布在我们的应用中计算出的百分位值。这些值在各个维度上都是不可凝聚的。

&emsp;&emsp;2.publishPercentileHistogram-用于发布直方图，该直方图适用于在Prometheus使用histogram_quantile，Atlas使用:percentile和Wavefront使用中计算可聚集的（跨维度）百分位近似hs()。对于Prometheus和Atlas，结果的直方图中的buckets值由MicroMeter根据由Netflix凭经验确定的生成器进行预设，以在大多数现实世界timer和分布式summary上产生合理的误差。默认情况下，生成器会产生276个存储桶，但Micrometer仅发送那些在设置范围内（包括minimumExpectedValue和maximumExpectedValue）的buckets值。MicroMeter默认将timer限制在1毫秒到1分钟的范围内，每个timer产生73个直方图桶。publishPercentileHistogram 对不支持可凝集百分位数逼近的系统没有影响-这些系统未提供直方图。

&emsp;&emsp;3.sla-发布包含我们SLA定义的buckets的累积直方图。与publishPercentileHistogram支持可聚集百分位数的监视系统配合使用，此设置将其他buckets添加到已发布的直方图中。在不支持可聚集百分位数的系统上使用时，此设置将导致仅使用这些存储桶发布直方图。

&emsp;&emsp;4.minimumExpectedValue/ maximumExpectedValue-控制publishPercentileHistogram发送的存储桶的数量，以及控制基础HdrHistogram结构的准确性和内存占用量。

&emsp;&emsp;由于将百分位数发送到监视系统会生成其他时间序列，因此通常最好不要在作为依赖项包含在应用程序中的核心库中对其进行配置。取而代之的是，应用程序可以通过meter过滤器为某些计时器/分配摘要集启用此行为。

&emsp;&emsp;例如，假设我们在一个公共库中有几个计时器。我们为这些计时器名称添加了前缀myservice：

    registry.timer("myservice.http.requests").record(..);
    
    registry.timer("myservice.db.requests").record(..);

&emsp;&emsp;我们可以通过仪表过滤器为两个timer打开客户端百分位数：

    registry.config().meterFilter(
	    new MeterFilter() {
	        @Override
	        public DistributionStatisticConfig configure(Meter.Id id, DistributionStatisticConfig config) {
	            if(id.getName().startsWith("myservice")) {
	                return DistributionStatisticConfig.builder()
	                    .percentiles(0.95)
	                    .build()
	                    .merge(config);
	            }
	            return config;
	        }
	    });