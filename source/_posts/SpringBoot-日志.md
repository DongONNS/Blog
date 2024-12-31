---
layout: posts
title: SpringBoot-日志
date: 2019-11-10 10:06:55
categories: SpringBoot
tags:
---

在我们的日常开发中，通常需要查看日志来帮我们解决程序运行中的一些错误问题及对程序运行过程的监控，这篇文章主要是讲解在SpringBoot中日志的工作原理、日志实现类的选择、日志的展示及存储方式等内容。

<!--more-->

# 一、常用的日志框架

JUL、JCL、Jboss-logging、logback、log4j、log4j2、slf4j....

其中属于**日志门面**（日志的抽象层）的是：JCL（Jakarta Commons Logging）    **SLF4j**（Simple Logging Facade for Java）、jboss-logging	

属于日志的**实现类**的是：Log4j2 **Logback**、Log4j JUL（java.util.logging） 

我们一般是选择SLF4j作为抽象层，Logback作为实现类

springboot也是选用的**SLF4j和Logback**

---

# 二、如何在系统中使用SLF4j

开发的时候，日志记录方法的调用，不应该来直接调用日志的实现类，而是调用日志抽象层里面的方法；给系统里面导入slf4j的jar和 logback的实现jar

具体的实现代码如下：

    import org.slf4j.Logger; 
    import org.slf4j.LoggerFactory;   

    public class HelloWorld {
       public static void main(String[] args) {
         Logger logger = LoggerFactory.getLogger(HelloWorld.class);     
    	 logger.info("Hello World");
       }
    }


>抽象类与实现类之间的关系如下所示,每一个日志的实现框架都有自己的配置文件。
>使用slf4j以后，配置文件还是做成日志实现框架自己本身的配置文件。

![抽象类与实现类的关系](https://wx1.sinaimg.cn/mw690/007857NYgy1g8sqnny7iaj30w00homz4.jpg)

---

# 三、遗留问题
在springboot中我们含有很多的框架，如Spring、Hebernate、MyBatis等，这些不同的日志框架使用的日志实现类是不相同的，那么我们如何实现不同的日志实现类之间的协调统一呢？

如：a（slf4j+logback）: Spring（commons-logging）、Hibernate（jboss-logging）、MyBatis、xxxx 统一日志记录，即使是别的框架和我一起统一使用slf4j进行输出？

**如何让系统中所有的日志都统一到slf4j：**
 
    1、将系统中其他日志框架先排除出去；
    
    2、用中间包来替换原有的日志框架；
     
    3、我们导入slf4j其他的实现；

![日志框架的替换类](https://wx4.sinaimg.cn/mw690/007857NYly1g8sr40230bj316e0u00xc.jpg)

---

# 四、SpringBoot的日志依赖关系：
![日志依赖关系](https://wx3.sinaimg.cn/mw690/007857NYgy1g8sr81yfcnj30o709f0ta.jpg)

总结：
 
    1）、SpringBoot底层也是使用slf4j+logback的方式进行日志记录
    
    2）、SpringBoot也把其他的日志都替换成了slf4j；
    
    3）、中间替换包？
    
    4）、如果我们要引入其他框架？一定要把这个框架的默认日志依赖移除掉？Spring框架用的是commons-logging；


如下为在SpringBoot中引入依赖后的**转换jar包**

![日志转换包](https://wx4.sinaimg.cn/mw690/007857NYgy1g8sreyk7njj30cp06odg2.jpg)

SpringBoot能自动适配所有的日志，而且底层使用slf4j+logback的方式记录日志，引入其他框架的时候，只需要 把这个框架依赖的日志框架排除掉即可；

    <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring‐core</artifactId>              
    		  <exclusions>
    			  <exclusion>
    					<groupId>commons‐logging</groupId>
    					<artifactId>commons‐logging</artifactId>
    			  </exclusion>
    		  </exclusions>
    </dependency 

---

# 五 日志的使用
SpringBoot默认帮我们配置好了日志；

## 默认配置

    import org.junit.jupiter.api.Test;
    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
    import org.springframework.boot.test.context.SpringBootTest;

    @SpringBootTest
    class DemoApplicationTests {
    //日志记录器
    Logger logger = LoggerFactory.getLogger(getClass());

    @Test
    void contextLoads() {
        //日志的级别由低到高 trace<debug<info<warn<error
        //可以调整输出的日志级别，日志就只会在这个级别以后的级别中生效
        logger.trace("这是trace()日志");
        logger.debug("这是debug日志");
        //springboot默认提供的是info级别
        logger.info("这是info日志");
        logger.warn("这是warn日志");
        logger.error("这是error日志");
    	}
    }

当然我们可以对日志的输出格式、输出位置、默认级别等等做相应的调整


    #指定日志的输出级别，默认条件为info级别的。
	logging.level.csu = trace

    #在当前目录下生成springboot.log日志
    #不指定目录则在当前目录下生成springboot.log日志
    #logging.file = springboot.log
    #也可以指定springboot.log的生成路径
    logging.file= F:/springboot.log

    #指定控制台输出的文件格式
    logging.pattern.console==%d{yyyy‐MM‐dd} [%thread] %‐5level %logger{50} ‐ %msg%n 

    #指定文件中日志的输出格式
    logging.pattern.file=%d{yyyy‐MM‐dd} === [%thread] === %‐5level === %logger{50} ==== %msg%n

关于**日志的输出格式简介**：

	日志输出格式： 

	%d表示日期时间 
	
	%thread表示线程名

	%‐5level：级别从左显示5个字符宽度          

	%logger{50} 表示logger名字长50个字符，否则按照句点分割。           

	%msg：日志消息，         

	%n是换行符

## 指定配置
给类路径下放上每个日志框架自己的配置文件即可；SpringBoot就不使用他默认配置的了

logback.xml：直接就被日志框架识别了； 

logback-spring.xml：日志框架就不直接加载日志的配置项，由SpringBoot解析日志配置，可以使用SpringBoot 的高级Proﬁle功能

所以我们一般使用加上了“-spring”后缀的命名格式，如下所示在开发过程中就可以指定在开发环境和非开发环境使用的日志格式了。

    <springProfile name="dev">                 
    	<pattern>%d{yyyy‐MM‐dd HH:mm:ss.SSS} ‐‐‐‐> [%thread] ‐‐‐> %‐5level  %logger{50} ‐ %msg%n</pattern>             
    </springProfile>             
    
    <springProfile name="!dev">                 
    	<pattern>%d{yyyy‐MM‐dd HH:mm:ss.SSS} ==== [%thread] ==== %‐5level  %logger{50} ‐ %msg%n</pattern>             
    </springProfile>

如果直接使用没有“-spring”后缀的格式设置开发环境和非开发环境的日志格式会发生错误；

当然我们也可以使用在配置中设置环境变量来进行开发环境选择：spring.profiles.active=dev  
![环境变量设置](https://wx3.sinaimg.cn/mw690/007857NYgy1g8sslk1dccj311j0ovjst.jpg)   

---

# 六 切换日志框架
SpringBoot的默认方式为SLF4j+Logback形式的，现在我们将其切换成SLF4j+log4j的形式，具体操作如下：


![切换框架1](https://wx1.sinaimg.cn/small/007857NYly1g8stgrfhm3j30kg0d70ti.jpg)

![切换框架2](https://wx2.sinaimg.cn/small/007857NYly1g8stgum4ucj30py0870tz.jpg)

![切换框架3](https://wx1.sinaimg.cn/mw690/007857NYly1g8stis21fpj30oa0dtta9.jpg)

![切换框架4](https://wx1.sinaimg.cn/mw690/007857NYly1g8stivul1dj30ry07zweu.jpg)










