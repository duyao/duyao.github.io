title: spring-ioc & aop
toc: true
date: 2017-03-18 18:26:58
tags: spring
categories: note
---
# spring知识点
http://jinnianshilongnian.iteye.com/blog/1482071
## Spring注入方式

Spring注入是指在启动Spring容器加载bean配置的时候，完成对变量的赋值行为
- 设值注入
- 构造注入

### 设值注入
通过set方法进行注入

`spring-injection.xml`
```
<bean id="injectionService" class="com.imooc.ioc.injection.service.InjectionServiceImpl"> -->
	<property name="injectionDAO" ref="injectionDAO"></property>
</bean>

<bean id="injectionDAO" class="com.imooc.ioc.injection.dao.InjectionDAOImpl"></bean>
```
`InjectionServiceImpl.java`
```
//设值注入
public void setInjectionDAO(InjectionDAO injectionDAO) {
	this.injectionDAO = injectionDAO;
}
```

### 构造注入
使用显示构造器来完成注入,构造器参数必须与配置文件的名字相同

`spring-injection.xml`
```
<bean id="injectionService" class="com.imooc.ioc.injection.service.InjectionServiceImpl">
	<constructor-arg name="injectionDAO" ref="injectionDAO"></constructor-arg>
</bean>
<bean id="injectionDAO" class="com.imooc.ioc.injection.dao.InjectionDAOImpl"></bean>
```
`InjectionServiceImpl.java`
```
//构造器注入
//public InjectionServiceImpl(InjectionDAO injectionDAO1)无法实现注入
public InjectionServiceImpl(InjectionDAO injectionDAO) {
	this.injectionDAO = injectionDAO;
}
```
## Bean的配置项
- Id：IOC容器中的唯一标示
- Class：具体事例化的哪一个类
- Scope：作用域，范围
- Constructor arguments：构造器参数
- Properties：属性
- Autowiring mode：自动装配模式
- lazy-initialization mode：懒加载模式
- Initialization/destruction method：初始化/销毁方法

## Bean作用域
- singleton：单例，一个Bean容器只存在一份，比如一个context中存在一份，**spring会缓存**
- prototype：每次请求都会创建新的实例，destroy方式不生效,**spring不会缓存**
- request：每次http请求创建一个实例且尽在当前request内有效
- session：同上，每次http请求创建，当前session内有效
- global session：基于portlet(portlet可以登录多个系统定义了global session)的web中有效

## Bean的生命周期
- 定义
- 初始化
- 使用
- 销毁

### 初始化

- 实现`import org.springframework.beans.factory.InitializingBean`接口，覆盖`afterPropertiesSet`方法
- 配置`init-method`

### 销毁
- 实现`import org.springframework.beans.factory.DisposableBean`接口，覆盖`destroy`方法
- 配置`destroy-method`

### 配置全局的初始化销毁方法
```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd"
        default-init-method="defautInit" default-destroy-method="defaultDestroy">
```

### 例子
`BeanLifeCycle.java`
```java
//实现了接口
public class BeanLifeCycle implements InitializingBean, DisposableBean {

	//全局配置
	public void defautInit() {
		System.out.println("Bean defautInit.");
	}

	public void defaultDestroy() {
		System.out.println("Bean defaultDestroy.");
	}

	//实现接口
	@Override
	public void destroy() throws Exception {
		System.out.println("Bean destroy.");
	}

	@Override
	public void afterPropertiesSet() throws Exception {
		System.out.println("Bean afterPropertiesSet.");
	}

	//配置文件
	public void start() {
		System.out.println("Bean start .");
	}

	public void stop() {
		System.out.println("Bean stop.");
	}

}
```
`spring-lifecycle.xml`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd"

        default-init-method="defautInit" default-destroy-method="defaultDestroy">

        <!--配置文件-->
        <bean id="beanLifeCycle" class="com.imooc.lifecycle.BeanLifeCycle"  
        init-method="start" destroy-method="stop"></bean>

 </beans>
```

### 总结
**优先级：实现接口 > 配置文件 > 全局配置**
在全局配置中的方法没有或者实现，其他两者不可以

## Aware
- Spring中提供了一些以`Aware`结尾的接口，实现了Aware接口的bean在被初始化后，可以获取相应的资源
- 通过Aware接口，可以对Spring相应资源进行操作
- 为对Spring进行简单的扩展提供了方便的入口

`MoocBeanName.java`
```java
public class MoocBeanName implements BeanNameAware, ApplicationContextAware {

	private String beanName;

	//实现BeanNameAware方法
	@Override
	public void setBeanName(String name) {
		this.beanName = name;
		System.out.println("MoocBeanName : " + name);
	}

	//实现ApplicationContextAware的方法
	@Override
	public void setApplicationContext(ApplicationContext applicationContext)
			throws BeansException {
		System.out.println("setApplicationContext : " + applicationContext.getBean(this.beanName).hashCode());
	}

}
```
`spring-aware.xml`
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd" >
	<bean id="moocBeanName" class="com.imooc.aware.MoocBeanName" ></bean>       
 </beans>
```

## Bean的自动装配(Autowiring)
不需要配置属性set或者constructor，完成Bean的注入

- No
不要任何操作
- byName
根据属性名自动装配。此选项经检查容器并根据名字查找与属性完全一致的bean，并将其与属性自动装配
- byType
如果容器中存在一个与指定属性类型相同的bean，那么将与该属性自动装配；
如果存在多个该类型的bean，那么抛出异常，并指出不能使用byType方式进行自动装配；
如果没有找到相匹配的bean，什么事也不发生
**配置bean时可以没有id**
- Constructor
与byType方式类似，不同之处在于它应用于构造器参数
如果没有找到与构造器参数类型一致的bean，那么抛出异常

### 例子
`spring-autowiring.xml`
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd"
        default-autowire="constructor">
        <!--default-autowire上面说的几种方法-->
        <bean id="autoWiringService" class="com.imooc.autowiring.AutoWiringService" ></bean>
        <bean class="com.imooc.autowiring.AutoWiringDAO" ></bean>
 </beans>
 ```
 `AutoWiringService.java`
 ```
 public class AutoWiringService {

	private AutoWiringDAO autoWiringDAO;

	//构造器注入
	public AutoWiringService(AutoWiringDAO autoWiringDAO) {
		System.out.println("AutoWiringService");
		this.autoWiringDAO = autoWiringDAO;
	}

	//set注入
	public void setAutoWiringDAO(AutoWiringDAO autoWiringDAO) {
		System.out.println("setAutoWiringDAO");
		this.autoWiringDAO = autoWiringDAO;
	}

	public void say(String word) {
		this.autoWiringDAO.say(word);
	}

}
```
`AutoWiringDAO.java`
```
public class AutoWiringDAO {

	public void say(String word) {
		System.out.println("AutoWiringDAO : " + word);
	}

}
```


# ioc原理
一篇记录读tiny-spring的文章
https://github.com/duyao/tiny-spring

https://www.zybuluo.com/dugu9sword/note/382745

https://my.oschina.net/flashsword/blog/192551
https://my.oschina.net/flashsword/blog/194481
## BeanFactory 的构造与执行

BeanFactory 的核心方法是 getBean(String) 方法，用于从工厂中取出所需要的 Bean 。AbstractBeanFactory 规定了基本的构造和执行流程。

getBean 的流程：包括实例化和初始化，也就是生成 Bean，再执行一些初始化操作。

1.doCreateBean ：实例化 Bean。
  a.createInstance ：生成一个新的实例。

  b.applyProperties ：注入属性，包括依赖注入的过程。在依赖注入的过程中，如果 Bean 实现了 BeanFactoryAware 接口，则将容器的引用传入到 Bean 中去，这样，Bean 将获取对容器操作的权限，也就允许了 编写扩展 IoC 容器的功能的 Bean。

2.initializeBean(bean) ： 初始化 Bean。
  a. 从 BeanPostProcessor 列表中，依次取出 BeanPostProcessor 执行 bean = postProcessBeforeInitialization(bean,beanName) 。（为什么调用 BeanPostProceesor 中提供方法时，不是直接 post...(bean,beanName) 而是 bean = post...(bean,beanName) 呢？见分析1 。另外，BeanPostProcessor 列表的获取有问题，见分析2。）
  b. 初始化方法（tiny-spring 未实现对初始化方法的支持）。
  c. 从 BeanPostProcessor 列表中， 依次取出 BeanPostProcessor 执行其 bean = postProcessAfterInitialization(bean,beanName)。

## ApplicationContext 的构造和执行

ApplicationContext 的核心方法是 refresh() 方法，用于从资源文件加载类定义、扩展容器的功能。

refresh 的流程：

总体来说，tiny-spring 的 ApplicaitonContext 使用流程是这样的：

1.loadBeanDefinitions(BeanFactory) ：
  类定义的读取和加载，并注入到内置的 BeanFactory 中，这里的可扩展性在于，未对加载方法进行要求，也就是可以从不同来源的不同类型的资源进行加载。

2.registerBeanPostProcessors(BeanFactory) ：
  获取所有的 BeanPostProcessor，并注册到 BeanFactory 维护的 BeanPostProcessor 列表去。

3.onRefresh ：
  a. preInstantiateSingletons ：以单例的方式，初始化所有 Bean。通过**主动调用 getBean ** （见上面get过程） 实例化、注入属性、然后初始化 BeanFactory 中所有的 Bean。由于所有的 BeanPostProcessor 都已经在第 2 步中完成实例化了，因此接下来实例化的是普通 Bean，因此普通 Bean 的初始化过程可以正常执行。


BeanPostProcessor是Spring容器的一个扩展点，可以进行自定义的实例化、初始化、依赖装配、依赖检查等流程，即可以覆盖默认的实例化，也可以增强初始化、依赖注入、依赖检查等流程




# AOP（Aspect Orient Programming）
AOP（Aspect Orient Programming），也就是面向方面编程，作为面向对象编程的一种补充，专门用于处理系统中分布于各个模块（不同方法）中的交叉关注点的问题，在 Java EE 应用中，常常通过 AOP 来处理一些具有横切性质的系统级服务，如事务管理、安全检查、缓存、对象池管理等。
AOP 实现的关键就在于 AOP 框架自动创建的 AOP 代理，AOP 代理主要分为静态代理(预编译)和动态代理(运行期间)两大类，静态代理以 AspectJ 为代表；而动态代理则以 Spring AOP 为代表。



## SpringAOP
与 AspectJ 相同的是，Spring AOP 同样需要对目标类进行增强，也就是生成新的 AOP 代理类；
与 AspectJ 不同的是，**Spring AOP 无需使用任何特殊命令对 Java 源代码进行编译，它采用运行时动态地、在内存中临时生成“代理类”的方式来生成 AOP 代理。**
Spring 允许使用 AspectJ Annotation 用于定义方面（Aspect）、切入点（Pointcut）和增强处理（Advice），Spring 框架则可识别并根据这些 Annotation 来生成 AOP 代理。Spring 只是使用了和 AspectJ 5 一样的注解，但**并没有使用 AspectJ 的编译器或者织入器（Weaver）** ，底层依然使用的是 Spring AOP，依然是在运行时动态生成 AOP 代理，并不依赖于 AspectJ 的编译器或者织入器。

简单地说，**Spring 依然采用运行时生成动态代理的方式来增强目标对象，所以它不需要增加额外的编译，也不需要 AspectJ 的织入器支持**；而 AspectJ 在采用编译时增强，所以 AspectJ 需要使用自己的编译器来编译 Java 文件，还需要织入器。

不是为了提供强大的aop服务(AspectJ)，而侧重于提供与spring相关的基本aop服务

https://www.ibm.com/developerworks/cn/java/j-lo-springaopcglib/#ibm-pcon

### 一些概念
![AOP概念](http://7xilc8.com1.z0.glb.clouddn.com/aop.png)

![Advice类型](http://7xilc8.com1.z0.glb.clouddn.com/advice-charater.png)
#### advice、pointcut、aspect
```xml
<!--这是一个切面，里面定义了切面要执行的逻辑，比如有after方法、before方法等-->
<bean id="moocAspect" class="com.imooc.aop.schema.advice.MoocAspect"></bean>
<!--这一个普通的bean，这个bean希望有切面注入来实现一些功能-->
<bean id="aspectBiz" class="com.imooc.aop.schema.advice.biz.AspectBiz"></bean>
<aop:config>
  <!--首先声明一个切面-->
	<aop:aspect id="moocAspectAOP" ref="moocAspect">
    <!--pointcut指出这个切面到底在哪里执行-->
			<aop:pointcut expression="execution(* com.imooc.aop.schema.advice.biz.*Biz.*(..))" id="moocPiontcut"/>
      <!--下面这些方法都是用来表示pointcut执行的时候到底使用的是切面的什么方法，在什么时候-->
			<aop:before method="before" pointcut-ref="moocPiontcut"/>
			<aop:after-returning method="afterReturning" pointcut-ref="moocPiontcut"/>
		  <aop:after-throwing method="afterThrowing" pointcut-ref="moocPiontcut"/>
			<aop:after method="after" pointcut-ref="moocPiontcut"/>
			<aop:around method="around" pointcut-ref="moocPiontcut"/>

			<aop:around method="aroundInit" pointcut="execution(* com.imooc.aop.schema.advice.biz.AspectBiz.init(String, int))
						and args(bizName, times)"/>
	</aop:aspect>
</aop:config>
```


#### Introduction
允许一个切面声明一个实现指定接口的通知对象，并且提供了一个接口实现类来代表这些对象
`	</aop:aspect>`中`<aop:declare-parents>`元素声明该元素用于声明所匹配的类型拥有一个新的parent
```xml
<aop:config>
		<aop:aspect id="moocAspectAOP" ref="moocAspect">
      <!--interface表示匹配者的新父接口，default-impl是父接口的实现类-->
    <aop:declare-parents types-matching="com.imooc.aop.schema.advice.biz.*(+)"
    	implement-interface="com.imooc.aop.schema.advice.Fit"
    	default-impl="com.imooc.aop.schema.advice.FitImpl"/>
    </aop:aspect>
</aop:config>
```
#### Advisor
advisor与aspect有相似的用法，通常与事务`<tx:advice>`一起使用
区别：**advisor只持有一个Pointcut和一个advice**，而aspect可以多个pointcut和多个advice
```xml
<tx:advice id="txAdvice" transaction-manager="transactionManager">
	<tx:attributes>
		<!--
			propagation	:事务传播行为
			isolation	:事务的隔离级别
			read-only	:只读
			rollback-for:发生哪些异常回滚
			no-rollback-for	:发生哪些异常不回滚
			timeout		:过期信息
		 -->
		<tx:method name="transfer" propagation="REQUIRED"/>
	</tx:attributes>
</tx:advice>

	<!-- 配置切面 -->
<aop:config>
	<!-- 配置切入点 -->
	<aop:pointcut expression="execution(* com.zs.spring.demo3.AccountService+.*(..))" id="pointcut1"/>
	<!-- 配置切面 -->
	<aop:advisor advice-ref="txAdvice" pointcut-ref="pointcut1"/>
</aop:config>
```
## AspectJ
AspectJ 是一个基于 Java 语言的 AOP 框架，提供了强大的 AOP 功能，其他很多 AOP 框架都借鉴或采纳其中的一些思想。
AspectJ 是 Java 语言的一个 AOP 实现，其主要包括两个部分：
第一个部分定义了如何表达、定义 AOP 编程中的语法规范，通过这套语言规范，我们可以方便地用 AOP 来解决 Java 语言中存在的交叉关注点问题；
另一个部分是工具部分，包括编译器、调试工具等。
AspectJ 是最早、功能比较强大的 AOP 实现之一，对整套 AOP 机制都有较好的实现，很多其他语言的 AOP 实现，也借鉴或采纳了 AspectJ 中很多设计。在 Java 领域，AspectJ 中的很多语法结构基本上已成为 AOP 领域的标准。

### spring对AspectJ的支持
`@AspectJ`类似于java注解
运行时仍然是存spring aop
#### 配置方式
`@AspectJ`切面使用`@Aspect`配置注解，该注解的类可以有方法和字段，可能含有切入点pointcut、通知advice和引入introduction
`@Aspect`不能通过类路径自动检测，因此需要使用`@Component`注解或者在xml中配置bean

步骤：
1、在xml中开启`<aop:aspectj-autoproxy></aop:aspectj-autoproxy>`
2、定义切面
```java
@Component
@Aspect
public class MoocAspect {

	@Pointcut("execution(* com.imooc.aop.aspectj.biz.*Biz.*(..))")
	public void pointcut() {}

	@Pointcut("within(com.imooc.aop.aspectj.biz.*)")
	public void bizPointcut() {}

	//直接用已经定义好的pointcut
	@Before("pointcut()")
	public void before() {
		System.out.println("Before.");
	}

	@Before("pointcut() && args(arg)")
	public void beforeWithParam(String arg) {
		System.out.println("BeforeWithParam." + arg);
	}

	@Before("pointcut() && @annotation(moocMethod)")
	public void beforeWithAnnotaion(MoocMethod moocMethod) {
		System.out.println("BeforeWithAnnotation." + moocMethod.value());
	}

	@AfterReturning(pointcut="bizPointcut()", returning="returnValue")
	public void afterReturning(Object returnValue) {
		System.out.println("AfterReturning : " + returnValue);
	}

	@AfterThrowing(pointcut="pointcut()", throwing="e")
	public void afterThrowing(RuntimeException e) {
		System.out.println("AfterThrowing : " + e.getMessage());
	}

	@After("pointcut()")
	public void after() {
		System.out.println("After.");
	}

	@Around("pointcut()")
	public Object around(ProceedingJoinPoint pjp) throws Throwable {
		System.out.println("Around 1.");
		Object obj = pjp.proceed();
		System.out.println("Around 2.");
		System.out.println("Around : " + obj);
		return obj;
	}

}
```

# 动态代理技术
动态代理最常见应用是AOP(面向切面编程)。通过AOP，我们能够地拿到我们的程序运行到某个节点时的方法、对象、入参、返回参数，并动态地在方法调用前后新添一些新的方法逻辑，来满足我们的新需求，比如日志记录等。
动态代理常见有两种方式：基于JDK的反射技术的动态代理（必须是接口）和基于CGLib(任何类都可以，不一定是接口)的动态代理。

## JDK使用反射技术创建动态代理
JDK创建动态代理的核心是java.lang.reflect.InvocationHandler接口和java.lang.reflect.Proxy类。
```java
//实现InvocationHandler接口
public class JdkDynamicAopProxy extends AbstractAopProxy implements InvocationHandler {

    public JdkDynamicAopProxy(AdvisedSupport advised) {
        super(advised);
    }

//重写方法
	@Override
	public Object getProxy() {
		return Proxy.newProxyInstance(getClass().getClassLoader(), advised.getTargetSource().getInterfaces(), this);
	}

//重写调用方法
	@Override
	public Object invoke(final Object proxy, final Method method, final Object[] args) throws Throwable {
		MethodInterceptor methodInterceptor = advised.getMethodInterceptor();
		if (advised.getMethodMatcher() != null
				&& advised.getMethodMatcher().matches(method, advised.getTargetSource().getTarget().getClass())) {
			return methodInterceptor.invoke(new ReflectiveMethodInvocation(advised.getTargetSource().getTarget(),
					method, args));
		} else {
			return method.invoke(advised.getTargetSource().getTarget(), args);
		}
	}

}

```

## CGLib动态代理基本原理
CGLib——Code Generation Library，它是一个动态字节代码生成库，基于asm。使用CGLib时需要导入asm相关的jar包。而asm又是何方神圣？

asm是一个java字节码操纵框架，它能被用来动态生成类或者增强既有类的功能。ASM 可以直接产生二进制 class 文件，也可以在类被加载入 Java 虚拟机之前动态改变类行为。Java class 被存储在严格格式定义的 .class文件里，这些类文件拥有足够的元数据来解析类中的所有元素：类名称、方法、属性以及 Java 字节码（指令）。ASM从类文件中读入信息后，能够改变类行为，分析类信息，甚至能够根据用户要求生成新类。

用JDK的动态代理来模拟实现了一个性能监控的例子，用到了JDK内置的反射技术和java.lang.reflect.Proxy代理类
通过例子我们发现，它**只能通过让被代理类实现代理接口的方式来生成代理**，而CGLib的区别在于**通过在程序运行时动态生成一个被代理类的子类的方式来完成代理**。

CGLib有几个核心类:
1. Enhancer 它用于动态生成被代理的类的子类。使用此类生成子类的前奏是指定被代理类和指定CallBack接口
2. CallBack：它是一个很关键的接口，我们常常通过CallBack接口来配置我们的拦截方法，
3. MethodInterceptor：是CallBack的实现类，他会拦截我们被代理类的所有方法，来实现自己的增强细节。比如做点日志记录，方法处理等，处理完后，还能通过MethodProxy重新调用拦截掉的方法。
4. MethodProxy：主要用于重新调用MethodInterceptor拦截掉的方法，是jdk反射包中Method的代理类。
5. CallbackFilter：一个Enhancer生成类可以指定多个Callback,这样我们可以设定条件过滤，让被代理类中不同的方法被调用时使用不同的CallBack来进行处理。
https://yq.aliyun.com/articles/29130?spm=5176.8091938.0.0.kpT4Os
https://yq.aliyun.com/articles/29131?spm=5176.8091938.0.0.kpT4Os


# 事务管理
## PlatformTransactionManager事务管理器

```java
TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException;
void commit(TransactionStatus status) throws TransactionException;
void rollback(TransactionStatus status) throws TransactionException;
```
![PlatformTransactionManager实现类](http://7xilc8.com1.z0.glb.clouddn.com/spring-datasource.png)

## TransactionDefinition事务定义信息
```java
//主要方法
int getPropagationBehavior();
int getIsolationLevel();
int getTimeout();
boolean isReadOnly();
String getName();
TIMEOUT_DEFAULT
```
### 事务隔离级别
//事务隔离级别
ISOLATION_DEFAULT//默认的隔离级别，默认值为后台所使用的数据库的隔离级别，比如mysql是rr
ISOLATION_READ_UNCOMMITTED
ISOLATION_READ_COMMITTED
ISOLATION_REPEATABLE_READ
ISOLATION_SERIALIZABLE

不同的事务隔离级别可能会引发的问题：
脏读：一个事务读取了另一个事务改写但还未提交的数据，如果这个数据被回滚，则读到的数据是无效的
不可重复读：在同一个事务中，多次读取同一个数据，而返回的结果是不一致的
幻读：一个事务读取了几行记录后，后一个事务插入一些记录，幻读就发生了。再后来的查询中，第一个事务就会发现原来有些没有的记录。


  | 脏读 | 不可重复读 | 幻读|
----|:------:|:-----:|:-----:|
未提交读 | Y  | Y | Y
提交读 | N  | Y | Y
可重复读 | N  | N |  Y
可串行化 | N  | N | N

### 事务的传播特性

事务的传播行为：解决业务层方法之间相互调用的问题
比如有一个serviceA需要使用dao1.x()和dao2.y(),serviceB需要使用dao1.y()和dao2.x(),serviceC需要使用serviceA和serviceB来完成业务，那么事务的传播就来解决serviceC中事务的行为的
![事务的传播特性](http://7xilc8.com1.z0.glb.clouddn.com/transaction_propagation.png)
其实主要分为3类：
第一类是支持支持当前事务，也就是说当前事务和新事务是在同一个事务中的
第二类是不和当前事务混在一起，各走各的
第三类是嵌套

## TransactionStatus事务状态信息

```java
boolean isNewTransaction();
boolean hasSavepoint();
void setRollbackOnly();
boolean isRollbackOnly();
void flush();
boolean isCompleted();
```
## 配置声明式事务
### xml方式
```xml
	<!-- 配置业务层类 -->
	<bean id="accountService" class="com.zs.spring.demo3.AccountServiceImpl">
		<property name="accountDao" ref="accountDao" />
	</bean>

	<!-- 配置DAO类(简化，会自动配置JdbcTemplate) -->
	<bean id="accountDao" class="com.zs.spring.demo3.AccountDaoImpl">
		<property name="dataSource" ref="dataSource" />
	</bean>

	<!-- ==================================使用XML配置声明式的事务管理,基于tx/aop=============================================== -->

	<!-- 配置事务管理器 -->
	<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource" />
	</bean>

	<!-- 配置事务的通知 -->
	<tx:advice id="txAdvice" transaction-manager="transactionManager">
		<tx:attributes>
			<!--
				propagation	:事务传播行为
				isolation	:事务的隔离级别
				read-only	:只读
				rollback-for:发生哪些异常回滚
				no-rollback-for	:发生哪些异常不回滚
				timeout		:过期信息
			 -->
       <tx:method name="transfer" propagation="REQUIRED" isolation="SERIALIZABLE" read-only="true"/>
		</tx:attributes>
	</tx:advice>

	<!-- 配置切面 -->
	<aop:config>
		<!-- 配置切入点 -->
		<aop:pointcut expression="execution(* com.zs.spring.demo3.AccountService+.*(..))" id="pointcut1"/>
		<!-- 配置切面 -->
		<aop:advisor advice-ref="txAdvice" pointcut-ref="pointcut1"/>
	</aop:config>

```
无需更改service的代码，就可以完成事务的操作
### 注解
```xml
<!-- ==================================4.使用注解配置声明式事务============================================ -->

<!-- 配置事务管理器 -->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
  <property name="dataSource" ref="dataSource" />
</bean>

<!-- 开启注解事务 -->
<tx:annotation-driven transaction-manager="transactionManager"/>
```
service中的方法前面加上注解
```java
@Transactional(propagation = Propagation.REQUIRED,isolation = Isolation.REPEATABLE_READ,readOnly = true)
public void myTransaction() {
  ...
  System.out.println("myTransaction...  " ));
}
```
# 一些问题
## spring为什么默认为单例
>When a bean is a singleton, only one shared instance of the bean will be managed and all requests for beans with an id or ids matching that bean definition will result in that one specific bean instance being returned.

spring中的单例和设计模式中的单例是不一样的，在spring中单例意味着每一个bean只有一个实例
一是我们不用每次创建bean，这样可以减小开销，提高性能
二是减少了对象创建和垃圾收集的时间

http://stackoverflow.com/questions/21828664/why-spring-bean-is-singleton-scope

## spring保证线程安全吗
不保证，spring的bean是单例的，但不一定是线程安全的，单例与线程安全无必然联系
spring中bean还是无状态的,ejb是两者都有。
Stateless无状态用单例Singleton模式，Stateful有状态就用原型Prototype模式。
Stateful 有状态是多线程编码的天敌，所以在开发中尽量用Stateless无状态，无状态是不变(immutable)模式的应用，有很多优点：不用管线程和同步的问题，如果值是不可变的，程序不用担心多个线程改变共享状态，所以可以避免线程竞争的bugs. 因为没有竞争，就不用用locks等机制，所以无状态的不变机制，也可以避免产生死锁现象。
有状态就是有数据存储功能。有状态对象(Stateful Bean)，就是有实例变量的对象，可以保存数据，是非线程安全的。在不同方法调用间不保留任何状态。
无状态就是一次操作，不能保存数据。无状态对象(Stateless Bean)，就是没有实例变量的对象.不能保存数据，是不变类，是线程安全的。
http://www.moosepp.top/2016/08/21/%E6%9C%89%E7%8A%B6%E6%80%81bean%E5%92%8C%E6%97%A0%E7%8A%B6%E6%80%81bean%E7%9A%84%E4%B8%80%E4%BA%9B%E7%90%86%E8%A7%A3/

我们知道在一般情况下，只有无状态的Bean才可以在多线程环境下共享，在Spring中，绝大部分Bean都可以声明为singleton作用域。
就是因为Spring对一些Bean（如RequestContextHolder、TransactionSynchronizationManager、LocaleContextHolder等）中非线程安全的“状态性对象”采用**ThreadLocal**进行封装，让它们也成为线程安全的“状态性对象”，因此有状态的Bean就能够以singleton的方式在多线程中正常工作了。
或者浅显的解决办法就是将多态bean的作用域由“singleton”变更为“prototype”。


How to Make/Design a Thread Safe "Object"?
无状态、不变、
Design your beans immutable: for example have no setters and only use constructor arguments to create a bean. There are other ways, such as Builder pattern, etc..
Design your beans stateless: for example a bean that does something can be just a function (or several). This bean in most cases can and should be stateless, which means it does not have any state, it only does things with function arguments you provide each time (on each invocation)


http://stackoverflow.com/questions/15745140/are-spring-objects-thread-safe
