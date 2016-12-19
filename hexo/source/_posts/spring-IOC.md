title: spring DI
date: 2015-10-18 11:39:18
tags: spring
categories: note
---

## Dependency Injection (DI)
>When writing a complex Java application, application classes should be as independent as possible of other Java classes to increase the possibility to reuse these classes and to test them independently of other classes while doing unit testing. Dependency Injection helps in gluing these classes together and same time keeping them independent.

<!--more-->

## XML based configuration file

### 注入属性
有3种方法

- 属性注入
	
```
<property name="services" ref="userServices"></property>
	<!-- 注入上面添加的属性，用ref进行指定 -->
	<property name="user" ref="user"></property>
	<!-- 注入其他属性，使用value -->
	<property name="nameList">
		<list>
		 <value>123</value>
		 <value>45</value>
		</list>
	</property>
```

- 使用`autowire`关键字

`<bean id="user" class="com.dy.springIOC.model.User" autowire="default">`

- 使用`constructor-arg`
	
```
<bean id="userAction" class="com.dy.springIOC.action.UserAction" scope="prototype" >
	<constructor-arg ref="userServices"></constructor-arg>
</bean>
```

### scope

>When defining a <bean> in Spring, you have the option of declaring a scope for that bean. For example, To force Spring to produce a new bean instance each time one is needed, you should declare the bean's scope attribute to be `prototype`. Similar way if you want Spring to return the same bean instance each time one is needed, you should declare the bean's scope attribute to be `singleton`(default).

例如
```<bean id="userAction" class="com.dy.springIOC.action.UserAction" scope="prototype">```


### example

`bean.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans.xsd">


	<bean id="user" class="com.dy.springIOC.model.User">
		<property name="id" value="000"></property>
		<property name="name" value="rerre"></property>
	</bean>
	
	<!-- 这里创建bean相当于new一个对象，scope默认是单例，当创建多个对象，每个的内容不用时，需要将scope设置为多例 -->
	<!-- name是action中的get和set方法的属性，ref是bean文件中配置的id -->
	<bean id="userAction" class="com.dy.springIOC.action.UserAction" scope="prototype">

		<property name="services" ref="userServices"></property>
		<!-- 注入上面添加的属性，用ref进行指定 -->
		<property name="user" ref="user"></property>
		<!-- 注入其他属性 -->
		<property name="nameList">
			<list>
			 <value>123</value>
			 <value>45</value>
			</list>
		</property>
	</bean>

	<bean id="userDao" class="com.dy.springIOC.dao.Dao">

	</bean>
	<bean id="userServices" class="com.dy.springIOC.service.Services">
		<property name="iDao" ref="userDao"></property>
	</bean>
</beans>
```
## 测试
- Introduction

	测试时使用`ClassPathXmlApplicationContext`,结束后要关闭

	还有可以使用`beanFactory`

- Differences 

	Bean Factory

	- Bean instantiation/wiring

	Application Context

	- Bean instantiation/wiring
	- Automatic BeanPostProcessor registration
	- Automatic BeanFactoryPostProcessor registration
	- Convenient MessageSource access (for i18n)
	- ApplicationEvent publication

- Conclusion

	**Application Context is better.**

### test example
`testSpring`
```java
@Test
public void test01() {
	
	//	private BeanFactory factory = new ClassPathXmlApplicationContext("bean.xml");
	ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("bean.xml");
	//UserAction userAction = factory.getBean("userAction", UserAction.class);
	UserAction userAction = context.getBean("userAction",UserAction.class);
	User user1 = new User("amy", "123");
	userAction.setUser(user1);
	userAction.add();

	//如果action不设置成为多例，那么多个action都添加的是相同的值
	//UserAction userAction2 = factory.getBean("userAction",UserAction.class);
	UserAction userAction2 = context.getBean("userAction",UserAction.class);
	User user2 = new User("456", "Lily");
	//userAction2.setUser(user2);
	userAction2.add();
	
	context.close();
	
}
```

## Annotation
不在`xml`中注入对象，而是在Java文件中通过`annotation`来完成注入

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.2.xsd">

<!-- Activates various annotations -->
<context:annotation-config></context:annotation-config>
<!-- Scans the classpath for annotated components that will be auto-registered as Spring beans. -->
<context:component-scan base-package="com.dy.springIOC"></context:component-scan>

</beans>
```

然后使用`@Service`,`@Repository`,`@Component`,`@Autowired`,`@Scope`,`@Controller`等在函数或者类前使用

`@Component`相当于bean中的`id`，但是实际使用在各个层次中使用其他annotoion来代理

在action中用`@Controller`，在service中用`@Service`代替，在DAO中用`@Repository`
`@Repository`，`@Controller`，`@Service`，`@Component`关系

![关系](http://7xilc8.com1.z0.glb.clouddn.com/Spring%20Annotations.png "http://7xilc8.com1.z0.glb.clouddn.com/Spring%20Annotations.png")

注入在`setXX`方法前面声明使用`@Resource`
声明多例或者单例使用`@Scope`

```java
@Controller("userAction")
@Scope("prototype")
public class UserAction {

	public UserAction() {
		// TODO Auto-generated constructor stub
	}
	private IServices services;
	
	public IServices getServices() {
		return services;
	}

	//依赖注入，使用Resource
	@Resource
	public void setServices(IServices services) {
		this.services = services;
	}
}
```



