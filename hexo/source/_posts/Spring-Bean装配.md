title: Spring Bean装配
date: 2016-02-17 10:44:55
tags: spring
---


## Bean的配置项
- Id 
IOC容器中的唯一标示
- Class 
具体事例化的哪一个类
- Scope
作用域，范围
- Constructor arguments
构造器参数
- Properties
属性
- Autowiring mode
自动装配模式
- lazy-initialization mode
懒加载模式
- Initialization/destruction method
初始化/销毁方法

## Bean作用域
- singleton 
单例，一个Bean容器只存在一份，比如一个context中存在一份
- prototype
每次请求都会创建新的实例，destroy方式不生效
- request
每次http请求创建一个实例且尽在当前request内有效
- session
同上，每次http请求创建，当前session内有效
- global session
基于portlet(portlet可以登录多个系统定义了global session)的web中有效

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
```
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
```
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
- 优先级：实现接口 > 配置文件 > 全局配置
- 在全局配置中的方法没有或者实现，其他两者不可以

## Aware
- Spring中提供了一些以`Aware`结尾的接口，实现了Aware接口的bean在被初始化后，可以获取相应的资源
- 通过Aware接口，可以对Spring相应资源进行操作
- 为对Spring进行简单的扩展提供了方便的入口

`MoocBeanName.java`
```
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

## Resources
针对资源文件的统一接口

- UrlResources
URL对应的资源，根据一个URL地址即可构建
- ClassPathResources
获取类路径下的资源文件
- FileSystemResources
获取文件系统里面的资源
- ServletContextResources
ServletContext封装的资源，用于访问ServletContext环境下的资源
- InputStreamResources
针对于输入封装刘的资源
- ByteArrayResources
针对于字节数组封装的资源

### ResourceLoader
All application contexts implement the ResourcesLoader interface, and therefore all application contexts may be used to obtain Resources instances.

前缀perfix

- classpath:
- file:
- http:
- (none)

### 例子
`spring-resource.xml`
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd" >    
    <bean  id="moocResource" class="com.imooc.resource.MoocResource" ></bean>
</beans>
```
`MoocResource.java`
```
public class MoocResource implements ApplicationContextAware  {
	
	private ApplicationContext applicationContext;
	
	@Override
	public void setApplicationContext(ApplicationContext applicationContext)
			throws BeansException {
		this.applicationContext = applicationContext;
	}
	
	public void resource() throws IOException {
//		Resource resource = applicationContext.getResource("classpath:config.txt");
//		Resource resource = applicationContext.getResource("file:C:\\resource\\config.txt");
//		Resource resource = applicationContext.getResource("url:https://github.com/duyao");
		Resource resource = applicationContext.getResource("config.txt");
		System.out.println(resource.getFilename());
		System.out.println(resource.contentLength());
	}

}
```



