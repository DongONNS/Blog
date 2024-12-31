---
title: springboot2——Metrics
date: 2019-11-13 20:49:11
tags:
categories: Monitor System
---

<!--more-->

# Metrics

&emsp;&emsp;Spring Boot Actuator为Micrometer提供了依赖项管理和自动配置，Micrometer是一种支持大量监视系统的应用程序指标展示，包括：AppOptics、Atlas、Datadog、Dynatrace、Elastic、Ganglia、Graphite、Humio、Influx、JMX、KairosDB、New Relic、Prometheus、SignalFx、Simple (in-memory)、StatsD、Wavefront。

&emsp;&emsp;要了解有关Micrometer功能的更多信息，请参阅其参考文档，特别是[概念](https://dongonns.github.io/2019/11/12/MicroMeter%E5%9F%BA%E7%A1%80%E6%A6%82%E5%BF%B5/)部分。

---

## 1 入门
&emsp;&emsp;Spring Boot自动配置组合MeterRegistry，并为其在类路径上找到的每个支持的实现向组合添加注册表。在micrometer-registry-{system}运行时类路径中具有依赖项足以让Spring Boot配置注册表。

&emsp;&emsp;大多数注册表具有共同的特征。例如，即使Micrometer注册表实现位于类路径中，您也可以禁用特定的注册表。例如，禁用Datadog：

    management.metrics.export.datadog.enabled=false


&emsp;&emsp;Spring Boot还会将任何自动配置的注册表添加到Metrics该类的全局静态复合注册表中，除非您明确告诉它不要：

    management.metrics.use-global-registry=false

&emsp;&emsp;您可以注册任意数量的MeterRegistryCustomizerbean来进一步配置注册表，例如在向注册表注册任何计量器(meter)之前应用通用标签：

    @Bean
    MeterRegistryCustomizer<MeterRegistry> metricsCommonTags() {
    	return registry -> registry.config().commonTags("region", "us-east-1");
    }

&emsp;&emsp;您可以通过更具体地了解通用类型，将自定义项应用于特定的注册表实现：

    @Bean
    MeterRegistryCustomizer<GraphiteMeterRegistry> graphiteMetricsNamingConvention() {
    	return registry -> registry.config().namingConvention(MY_CUSTOM_CONVENTION);
    }

&emsp;&emsp;完成该设置后，您可以注入MeterRegistry组件并注册指标：

    @Component
    public class SampleBean {

	    private final Counter counter;
	
	    public SampleBean(MeterRegistry registry) {
	        this.counter = registry.counter("received.messages");
    }

    public void handleMessage(String message) {
        this.counter.increment();
        // handle message implementation
    	}
    }

&emsp;&emsp;Spring Boot还配置了[内置工具](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#production-ready-metrics-meter)（即MeterBinder实现），您可以通过配置或专用注释标记来控制它们。

## 2 支持的监控系统--Prometheus

&emsp;&emsp;Prometheus希望抓取或轮询单个应用程序实例以获取指标。Spring Boot提供了一个可用的执行器端点/actuator/prometheus以适当的格式显示Prometheus抓取。

&emsp;&emsp;端点默认情况下不可用，必须公开，有关更多详细信息，请参见[暴露端点](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#production-ready-endpoints-exposing-endpoints)。

&emsp;&emsp;这是scrape_config要添加到的示例prometheus.yml：


    scrape_configs:
      - job_name: 'spring'
	    metrics_path: '/actuator/prometheus'
	    static_configs:
      - targets: ['HOST:PORT']

&emsp;&emsp;对于短暂的或批处理的工作，其时间可能不够长，无法被抓取，可以使用Prometheus Pushgateway支持将其指标暴露给Prometheus。要启用Prometheus Pushgateway支持，请在项目中添加以下依赖项：

    <dependency>
	    <groupId>io.prometheus</groupId>
	    <artifactId>simpleclient_pushgateway</artifactId>
    </dependency>

&emsp;&emsp;当在类路径中存在Prometheus Pushgateway依赖项时，Spring Boot会自动配置PrometheusPushGatewayManager组件。这可以管理将指标推送到Prometheus Pushgateway。PrometheusPushGatewayManager可以在使用属性management.metrics.export.prometheus.pushgateway配置。对于高级配置，您还可以提供自己的PrometheusPushGatewayManager组件。

## 3 支持的指标

&emsp;&emsp;如果适用，Spring Boot将注册以下核心指标：



- JVM指标，报告以下各项的利用率：

&emsp;&emsp;&emsp;&emsp;各种内存和缓冲池

&emsp;&emsp;&emsp;&emsp;与垃圾收集有关的统计数据

&emsp;&emsp;&emsp;&emsp;线程利用率

&emsp;&emsp;&emsp;&emsp;加载/卸载的类数



- CPU指标

- 文件描述符指标

- 卡夫卡消耗指标

- Log4j2指标：记录每个级别记录到Log4j2的事件数

- Logback指标：记录每个级别记录到Logback的事件数

- 正常运行时间指标：报告正常运行时间的量度和代表应用程序绝对启动时间的固定量度

- Tomcat指标（server.tomcat.mbeanregistry.enabled必须设置true为才能注册所有Tomcat指标）


- [Spring集成指标](https://docs.spring.io/spring-integration/docs/5.2.1.RELEASE/reference/html/system-management.html#micrometer-integration)

### Spring MVC指标

&emsp;&emsp;通过自动配置，可以检测由Spring MVC处理的请求。当management.metrics.web.server.request.autotime.enabled为true时，将对所有请求进行这种检测。或者，当设置为false时，可以通过添加@Timed到请求处理方法来启用检测：

    @RestController
    @Timed 
    public class MyController {

	    @GetMapping("/api/people")
	    @Timed(extraTags = { "region", "us-east-1" }) 
	    @Timed(value = "all.people", longTask = true) 
	    public List<Person> listPeople() { ... }

    }

&emsp;&emsp;1.控制器类，用于对控制器中的每个请求处理程序启用计时。

&emsp;&emsp;2.一种启用单个端点的方法。如果您将它放在类中，则不必这样做，但是可以用于进一步为此特定端点自定义timer计时器。

&emsp;&emsp;3.一种longTask = true用于为该方法启用长任务计时器的方法。长任务计时器需要一个单独的度量metric名称，并且可以与短任务计时器堆叠在一起。

&emsp;&emsp;默认情况下，指标名称为http.server.requests。可以通过设置management.metrics.web.server.request.metric-name属性来自定义名称。

&emsp;&emsp;默认情况下，与Spring MVC相关的指标带有以下信息标记：

|标签|描述|
|:------:|:------:|
|exception|处理请求时引发的任何异常的简单类名。|
|method|请求的方法（例如GET或POST）|
|outcome|基于响应状态码的请求结果。1xx是INFORMATIONAL，2xx是SUCCESS，3xx是REDIRECTION，4xx CLIENT_ERROR和5xx是SERVER_ERROR|
|status|响应的HTTP状态代码（例如200或500）|
|uri|变量替换之前的请求URI模板（如果可能）（例如，/api/person/{id}）|

&emsp;&emsp;要自定义标签，请提供@Bean实现的WebMvcTagsProvider。


## HTTP客户端指标

&emsp;&emsp;启动器同时管理仪表RestTemplate和WebClient。为此，您必须注入自动配置的构建器并使用它来创建实例：

- RestTemplateBuilder 对于 RestTemplate

- WebClient.Builder 对于 WebClient

&emsp;&emsp;也可以手动应用负责此工具的定制程序，即MetricsRestTemplateCustomizer和MetricsWebClientCustomizer。

&emsp;&emsp;默认情况下，指标名称为http.client.requests。可以通过设置management.metrics.web.client.request.metric-name属性来自定义名称。

&emsp;&emsp;默认情况下，通过检测的客户端生成的指标会标记以下信息：

|标签|描述|
|:----------:|:-----------:|
|clientName|URI的主机部分|
|method|请求的方法（例如GET或POST）|
|outcome|基于响应状态码的请求结果。1xx是INFORMATIONAL，2xx是SUCCESS，3xx是REDIRECTION，4xx CLIENT_ERROR和5xx是SERVER_ERROR，否则UNKNOWN|
|status|响应的HTTP状态代码（例如200或500）（IO_ERROR如果存在I / O问题），否则CLIENT_ERROR|
|uri|变量替换之前的请求URI模板（如果可能）（例如，/api/person/{id}）|

&emsp;&emsp;要自定义标签，并根据您选择的客户端，可以提供@Bean实现RestTemplateExchangeTagsProvider或WebClientExchangeTagsProvider。RestTemplateExchangeTags和WebClientExchangeTags中有方便的静态函数。

### 缓存指标

&emsp;&emsp;通过自动配置可以在启动时使用前缀为Cache的指标来检测所有可用的cache。对于解百纳的cache信息是进行了标准化设置的。此外定制的cache指标也是可用的。

&emsp;&emsp;支持以下缓存库：

- Caffeine

- EhCache 2

- Hazelcast

- Any compliant JCache (JSR-107) 的实现

&emsp;&emsp;指标由cache的名称以及CacheManager从Bean名称派生的的名称标记(添加tag)。

&emsp;&emsp;只有启动时可用的缓存才绑定到注册表。对于在启动阶段后即时或以编程方式创建的缓存，需要显式注册。可以使用CacheMetricsRegistrarBean来简化该过程。

### 数据源指标

&emsp;&emsp;自动配置可使前缀为jdbc.connections的metric指标的所有DataSource对象可视化。数据源检测产生的gauge表示池中当前活动，空闲，最大允许和最小允许的连接。

&emsp;&emsp;metrics还通过DataSource基于bean名称的计算名称来标记（添加tag）。

&emsp;&emsp;默认情况下，Spring Boot为所有支持的数据源提供元数据。如果您喜欢的数据源不支持开箱即用，则可以添加其他DataSourcePoolMetadataProvider组件。请参阅DataSourcePoolMetadataProvidersConfiguration示例。

&emsp;&emsp;此外，Hikari特定指标带有hikaricp前缀。每个度量标准都以“池”的名称标记（可以通过来控制spring.datasource.name）。

## 4 注册自定义指标

&emsp;&emsp;要注册自定义指标，请插入MeterRegistry到您的组件中，如以下示例所示：

    class Dictionary {

	    private final List<String> words = new CopyOnWriteArrayList<>();
	
	    Dictionary(MeterRegistry registry) {
	        registry.gaugeCollectionSize("dictionary.size", Tags.empty(), this.words);
	    }
	
	    // …

    }

&emsp;&emsp;如果发现您反复在组件或应用程序中检测一组metrics，则可以将此组metrics封装在MeterBinder实现中。默认情况下，所有MeterBinderbean的metrics都将自动绑定到Spring-managed MeterRegistry。

## 5 自定义单个指标

&emsp;&emsp;如果需要将自定义应用于特定Meter实例，则可以使用该io.micrometer.core.instrument.config.MeterFilter接口。默认情况下，所有MeterFilter组件都会自动应用于MicroMeter的MeterRegistry.Config。

&emsp;&emsp;例如，如果要将所有仪表ID中以com.example开头的mytag.region标签重命名为mytag.area，则可以执行以下操作：


    @Bean

    public MeterFilter renameRegionTagMeterFilter() {
    	return MeterFilter.renameTag("com.example", "mytag.region", "mytag.area");
    }
 
### 通用标签
&emsp;&emsp;通用标签通常用于在操作环境，如主机，实例，区域，堆栈等，进行维度深入分析。通用标签适用于所有仪表，并可以按以下示例所示进行配置：

    management.metrics.tags.region=us-east-1
    management.metrics.tags.stack=prod

&emsp;&emsp;上面的示例将us-east-1和prod分别添加到region和stack到所有仪表。

如果使用Graphite，则常用标签的顺序很重要。由于使用这种方法不能保证通用标签的顺序，因此建议Graphite用户定义一个自定义MeterFilter。


### Per-meter属性

Per-meter适用于以给定名称开头的所有meter ID。例如，以下将禁用所有ID以example.remote开头的meter。

    management.metrics.enable.example.remote=false

&emsp;&emsp;以下属性允许per-meter自定义：

表8.per-meter自定义

|属性        |描述        |
|:----------:|:---------:|
|management.metrics.enable|是否拒绝仪表发出任何指标|
|management.metrics.distribution.percentiles-histogram|是否发布适合于计算可凝集（跨维度）百分位数逼近的直方图|
|management.metrics.distribution.minimum-expected-value，management.metrics.distribution.maximum-expected-value|通过限制期望值的范围来发布较少的直方图bucket|
|management.metrics.distribution.percentiles|发布在应用程序中计算的百分位值|
|management.metrics.distribution.sla|发布包含您的SLA定义的存储桶的累积直方图|

&emsp;&emsp;有关percentiles-histogram，percentiles并sla概念更多的细节，请参阅“[Histogram and percentiles](https://micrometer.io/docs/concepts#_histograms_and_percentiles)”部分文档。

## 6 指标端点

&emsp;&emsp;Spring Boot提供了一个metrics端点，可用于诊断检查应用程序收集的指标。端点默认情况下不可用，必须公开，有关更多详细信息，请参见[暴露端点](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#production-ready-endpoints-exposing-endpoints)。

&emsp;&emsp;路径 /actuator/metrics 展示可用meter名称的列表。您可以通过提供特定的meter名称作为选择器来向下浏览以查看有关该仪表的信息,例如:/actuator/metrics/jvm.memory.max。

&emsp;&emsp;您在此处使用的名称应与代码中使用的名称相匹配，而不是已经针对监视系统进行了命名约定标准化后的名称。

&emsp;&emsp;您还可以tag=KEY:VALUE在URL的末尾添加任意数量的查询参数，以在维度上进一步细分meter，例如/actuator/metrics/jvm.memory.max?tag=area:nonheap。

&emsp;&emsp;报告的测量值是与仪表名称和已应用的任何标签相匹配的所有仪表的统计信息的总和。因此，在上面的示例中，返回的“值”统计量是堆的“代码缓存”，“压缩类空间”和“元空间”区域的最大内存占用量的总和。如果您只想查看“ Metaspace”的最大大小，则可以添加一个额外的tag=id:Metaspace，即/actuator/metrics/jvm.memory.max?tag=area:nonheap&tag=id:Metaspace。
