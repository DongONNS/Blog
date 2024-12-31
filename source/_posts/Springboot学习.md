---
title: Springboot学习
date: 2019-11-16 22:17:43
tags: 
categories: springboot
---

<!-- more -->

## @PoatConstruct有什么作用

PostConstruct注释用于在完成依赖项注入以执行任何初始化之后需要执行的方法。必须在类投入使用之前调用此方法。

<!--more-->

所有支持依赖注入的类都必须支持此注释。即使类没有请求注入任何资源，也必须调用使用PostConstruct注释的方法。

只有一个方法可以使用此批注进行批注。

应用PostConstruct注释的方法必须满足以下所有条件：除了拦截器之外，方法绝不能有任何参数，在这种情况下它采用Interceptor规范定义的InvocationContext对象。

在拦截器类上定义的方法必须具有以下签名之一：

void <METHOD>（InvocationContext）Object <METHOD>（InvocationContext）抛出异常注意：
PostConstruct拦截器方法不能抛出应用程序异常，但可以声明它抛出检查异常，包括java.lang.Exception，

如果相同的拦截器方法除了生命周期事件之外插入业务或超时方法。

如果PostConstruct拦截器方法返回一个值，容器将忽略它。

在非拦截器类上定义的方法必须具有以下签名：void <METHOD>（）应用PostConstruct的方法可以是public，protected，package private或private。

除应用程序客户端外，该方法绝不能是静态的。

该方法可能是最终的。如果该方法抛出一个未经检查的异常，那么该类绝不能投入使用，除非EJB可以处理异常甚至从它们恢复的EJB。

### 构造方法 > @Autowired > @PostConstruct

1、从Java EE5规范开始，Servlet中增加了两个影响Servlet生命周期的注解，@PostConstruct和@PreDestroy，这两个注解被用来修饰一个非静态的void（）方法。写法有如下两种方式：

@PostConstruct

public void someMethod(){}

或者

public @PostConstruct void someMethod(){}

被@PostConstruct修饰的方法会在服务器加载Servlet的时候运行，并且只会被服务器执行一次。PostConstruct在构造函数之后执行，init（）方法之前执行。PreDestroy（）方法在destroy（）方法知性之后执行

执行顺序

#### 另外，spring中Constructor、@Autowired、@PostConstruct的顺序

#### 其实从依赖注入的字面意思就可以知道，要将对象p注入到对象a，那么首先就必须得生成对象a和对象p，才能执行注入。所以，如果一个类A中有个成员变量p被@Autowried注解，那么@Autowired注入是发生在A的构造方法执行完之后的。

#### 如果想在生成对象时完成某些初始化操作，而偏偏这些初始化操作又依赖于依赖注入，那么久无法在构造函数中实现。为此，可以使用@PostConstruct注解一个方法来完成初始化，@PostConstruct注解的方法将会在依赖注入完成后被自动调用。

#### Constructor >> @Autowired >> @PostConstruct

---

## @Aspect的作用是什么？

AOP为Aspect Oriented Programming的缩写，意为：面向切面编程，通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术.AOP是OOP的延续，是软件开发中的一个热点，也是Spring框架中的一个重要内容，是函数式编程的一种衍生范型。利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。

在spring AOP中业务逻辑仅仅只关注业务本身，将日志记录，性能统计，安全控制，事务处理，异常处理等代码从业务逻辑代码中划分出来，通过对这些行为的分离，我们希望可以将它们独立到非指导业务逻辑的方法中，进而改变这些行为的时候不影响业务逻辑的代码。


	@Aspect:作用是把当前类标识为一个切面供容器读取
	 
	@Pointcut：Pointcut是植入Advice的触发条件。每个Pointcut的定义包括2部分，一是表达式，二是方法签名。方法签名必须是 public及void型。可以将Pointcut中的方法看作是一个被Advice引用的助记符，因为表达式不直观，因此我们可以通过方法签名的方式为 此表达式命名。因此Pointcut中的方法只需要方法签名，而不需要在方法体内编写实际代码。
	
	@Around：环绕增强，相当于MethodInterceptor
	
	@AfterReturning：后置增强，相当于AfterReturningAdvice，方法正常退出时执行
	
	@Before：标识一个前置增强方法，相当于BeforeAdvice的功能，相似功能的还有
	
	@AfterThrowing：异常抛出增强，相当于ThrowsAdvice
	
	@After: final增强，不管是抛出异常或者正常退出都会执行



Spring AOP面向切面编程，可以用来配置事务、做日志、权限验证、在用户请求时做一些处理等等。用@Aspect做一个切面，就可以直接实现。

1.首先定义一个切面类，加上@Component  @Aspect这两个注解   

	@Component
	@Aspect
	public class LogAspect {
	
	private static final Logger logger = LoggerFactory.getLogger(LogAspect.class);
	private static final ObjectMapper OBJECT_MAPPER = new ObjectMapper();


	}
2.定义切点     

	private final String POINT_CUT = "execution(public * com.xhx.springboot.controller.*.*(..))";
	
	@Pointcut(POINT_CUT)
	public void pointCut(){}
切点表达式中，..两个点表明多个，*代表一个，  上面表达式代表切入com.xhx.springboot.controller包下的所有类的所有方法，方法参数不限，返回类型不限。  其中访问修饰符可以不写，不能用*，，第一个*代表返回类型不限，第二个*表示所有类，第三个*表示所有方法，..两个点表示方法里的参数不限。  然后用@Pointcut切点注解，想在一个空方法上面，一会儿在Advice通知中，直接调用这个空方法就行了，也可以把切点表达式卸载Advice通知中的，单独定义出来主要是为了好管理。

3.Advice，通知增强，主要包括五个注解Before,After,AfterReturning,AfterThrowing,Around，下面代码中关键地方都有注释，我都列出来了。

	@Before  在切点方法之前执行
	
	@After  在切点方法之后执行
	
	@AfterReturning 切点方法返回后执行
	
	@AfterThrowing 切点方法抛异常执行
	
	@Around 属于环绕增强，能控制切点执行前，执行后，，用这个注解后，程序抛异常，会影响@AfterThrowing这个注解