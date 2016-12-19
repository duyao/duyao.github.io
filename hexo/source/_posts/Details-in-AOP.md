title: Details in AOP
date: 2015-10-19 22:46:51
tags: spring
categories: note
---


## `execution` expression of `pointcut`
Spring AOP users are likely to use the execution pointcut designator the most often. The format of an execution expression is:

`execution(modifiers-pattern? ret-type-pattern declaring-type-pattern? name-pattern(param-pattern)
            throws-pattern?)`
All parts except the returning type pattern (ret-type-pattern in the snippet above), name pattern, and parameters pattern are optional. The returning type pattern determines what the return type of the method must be in order for a join point to be matched. Most frequently you will use `*` as the returning type pattern, which matches any return type. A fully-qualified type name will match only when the method returns the given type. The name pattern matches the method name. 

You can use the `*` wildcard as all or part of a name pattern. 

The parameters pattern is slightly more complex:
- `()` matches a method that takes no parameters
- `(..)` matches any number of parameters (zero or more)
- `(*)` matches a method taking one parameter of any type, 
- `(*,String)` matches a method taking two parameters, the first can be of any type, the second must be a String. 

<!--more-->

Some examples of common pointcut expressions are given below.

the execution of any public method:
`execution(public * *(..))`

- the execution of any method with a name beginning with "set":
`execution(* set*(..))`
- the execution of any method defined by the AccountService interface:
`execution(* com.xyz.service.AccountService.*(..))`
- the execution of any method defined in the service package:
`execution(* com.xyz.service.*.*(..))`
- the execution of any method defined in the service package or a sub-package:
`execution(* com.xyz.service..*.*(..))`


## `@Around` of `advice`
Around advice is declared using the `@Around` annotation. 

The first parameter of the advice method must be of type `ProceedingJoinPoint`. 
Within the body of the advice, calling `proceed()` on the ProceedingJoinPoint causes the underlying method to execute. 
The `proceed` method may also be called passing in an `Object[]` - the values in the array will be used as the arguments to the method execution when it proceeds.


## XML Schema based

`bean.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.2.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.2.xsd">


<context:component-scan base-package="com.dy"></context:component-scan>
<aop:aspectj-autoproxy/>

<aop:config >
	<aop:aspect id="myLogAspect" ref="logAspect">
		<aop:pointcut id="logPointCut"  expression="execution(* com.dy.springIOC.dao.*.add*(..))
											||execution(* com.dy.springIOC.dao.*.delete*(..))
											||execution(* com.dy.springIOC.dao.*.load*(..))" />
		<aop:before method="logStart" pointcut-ref="logPointCut"/>
		<aop:after method="logEnd" pointcut-ref="logPointCut"/>
		<aop:around method="aroundLog" pointcut-ref="logPointCut"/>
	</aop:aspect>
</aop:config>
	

</beans>
```

`LogAspect.java`

```java
@Component("logAspect")
@Aspect//声明这是一个切面
public class LogAspect {
	
	public void logStart(){
		
		Logger.info("开始记录日志");
	}
	

	public void logEnd(){
		Logger.info("结束日志");
	}

	
	public Object aroundLog(ProceedingJoinPoint joinPoint) throws Throwable{
		System.out.println("around before");
		Object out = joinPoint.proceed();
		System.out.println("around after");
	
		return out;
	}
}


```


## @AspectJ based

`bean.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.2.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.2.xsd">


<context:component-scan base-package="com.dy"></context:component-scan>
<aop:aspectj-autoproxy/>

	

</beans>

```


`LogAspect.java`

```java
@Component("logAspect")
@Aspect//声明这是一个切面
public class LogAspect {
	
	//声明@Pointcut后，后面的advice都可以使用，不用重复写
	//@Pointcut返回值为void，且函数为空
	@Pointcut("execution(* com.dy.springIOC.dao.*.add*(..))"
				+"||execution(* com.dy.springIOC.dao.*.delete*(..))"
				+"||execution(* com.dy.springIOC.dao.*.load*(..))")
	public void selectAll(){}//signature
	
	
	@Before("selectAll()")
//	@Before("execution(* com.dy.springIOC.dao.*.add*(..))"
//			+"||execution(* com.dy.springIOC.dao.*.delete*(..))"
//			+"||execution(* com.dy.springIOC.dao.*.load*(..))")
	public void logStart(){
		
		Logger.info("开始记录日志");
	}
	
//	@After("execution(* com.dy.springIOC.dao.*.add*(..))"
//			+"||execution(* com.dy.springIOC.dao.*.delete*(..))"
//			+"||execution(* com.dy.springIOC.dao.*.load*(..))")
	@After("selectAll()")
	public void logEnd(){
		Logger.info("结束日志");
	}

	/**
	 * 传入参数必须是ProceedingJoinPoint，返回值必须是Object
	 * 执行proceed方法得到返回的对象
	 * @throws Throwable 
	 */
	@Around("selectAll()")
//	public void aroundLog(){
//		Logger.info("around Log");
//	}
	public Object aroundLog(ProceedingJoinPoint joinPoint) throws Throwable{
		System.out.println("around before");
		Object out = joinPoint.proceed();
		System.out.println("around after");
	
		return out;
	}
```