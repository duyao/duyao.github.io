title: web开发
toc: true
date: 2017-03-20 08:48:08
tags:
- springmvc
- mybatis
- velocity
categories: note
---
# springmvc
springmvc主要是`DipatcherServlet`完成的。
`DipatcherServlet`其本身是一个Servlet,他继承了HttpServlet，因此在servlet启动时调用init方法就可以把`DipatcherServlet`启动起来。
`DipatcherServlet`在启动时做了很多事情，主要的是初始化了`HandlerMappings`映射关系和`ViewResolvers`解析页面、`HandlerAdapters`实现业务逻辑的handler实例对象

在`HandlerAdapters`中可以得到`ModelAndView`。
`ModelAndView`是view和handler实例对象的纽带
`ViewResolvers`可以完成设置页面渲染比如是freemaker或者velocity

http://jinnianshilongnian.iteye.com/blog/1617451
## 工作过程
![springmvc工作过程](http://7xilc8.com1.z0.glb.clouddn.com/springmvc1.png)
![springmvc工作过程](http://7xilc8.com1.z0.glb.clouddn.com/springmvc.jpg)
工作原理:
(在HandlerMapping以前做的事情：1、文件上传解析，如果请求类型是multipart将通过MultipartResolver进行文件上传解析；)
1、客户端发出一个http请求给web服务器，web服务器对http请求进行解析，如果匹配DispatcherServlet的请求映射路径（在web.xml中指定），web容器将请求转交给DispatcherServlet.

2、DipatcherServlet接收到这个请求之后将根据请求的信息（包括URL、Http方法、请求报文头和请求参数Cookie等）以及HandlerMapping的配置找到处理请求的处理器（Handler）。

3-4、DispatcherServlet根据HandlerMapping找到对应的Handler,将处理权交给Handler（Handler将具体的处理进行封装），再由具体的HandlerAdapter对Handler进行具体的调用。

5、Handler对数据处理完成以后将返回一个ModelAndView()对象给DispatcherServlet。

6、Handler返回的ModelAndView()只是一个逻辑视图并不是一个正式的视图，DispatcherSevlet通过ViewResolver将逻辑视图转化为真正的视图View。

7、Dispatcher通过model解析出ModelAndView()中的参数进行解析最终展现出完整的view并返回给客户端。


## DispatcherServlet中使用的特殊的Bean
DispatcherServlet默认使用WebApplicationContext作为上下文，因此我们来看一下该上下文中有哪些特殊的Bean：
1、Controller：处理器/页面控制器，做的是MVC中的C的事情，但控制逻辑转移到前端控制器了，用于对请求进行处理；
2、HandlerMapping：请求到处理器的映射，如果映射成功返回一个HandlerExecutionChain对象（包含一个Handler处理器（页面控制器）对象、多个HandlerInterceptor拦截器）对象；如BeanNameUrlHandlerMapping将URL与Bean名字映射，映射成功的Bean就是此处的处理器；
3、HandlerAdapter：HandlerAdapter将会把处理器包装为适配器，从而支持多种类型的处理器，即适配器设计模式的应用，从而很容易支持很多类型的处理器；如SimpleControllerHandlerAdapter将对实现了Controller接口的Bean进行适配，并且掉处理器的handleRequest方法进行功能处理；
4、ViewResolver：ViewResolver将把逻辑视图名解析为具体的View，通过这种策略模式，很容易更换其他视图技术；如InternalResourceViewResolver将逻辑视图名映射为jsp视图；
5、LocalResover：本地化解析，因为Spring支持国际化，因此LocalResover解析客户端的Locale信息从而方便进行国际化；
6、ThemeResovler：主题解析，通过它来实现一个页面多套风格，即常见的类似于软件皮肤效果；
7、MultipartResolver：文件上传解析，用于支持文件上传；
8、HandlerExceptionResolver：处理器异常解析，可以将异常映射到相应的统一错误界面，从而显示用户友好的界面（而不是给用户看到具体的错误信息）；
9、RequestToViewNameTranslator：当处理器没有返回逻辑视图名等相关信息时，自动将请求URL映射为逻辑视图名；
10、FlashMapManager：用于管理FlashMap的策略接口，FlashMap用于存储一个请求的输出，当进入另一个请求时作为该请求的输入，通常用于重定向场景，后边会细述。

http://jinnianshilongnian.iteye.com/blog/1602617
更多资料
http://jinnianshilongnian.iteye.com/blog/1752171

## 配置文件
在`web.xml`中配置springmvc和spring的applicationcontext
```
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.4" xmlns="http://java.sun.com/xml/ns/j2ee"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee
	http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd">
  <display-name>Spring MVC Study</display-name>
  <!-- Spring应用上下文， 理解层次化的ApplicationContext -->
  <context-param>
 		<param-name>contextConfigLocation</param-name>
		<param-value>/WEB-INF/configs/spring/applicationContext*.xml</param-value>
  </context-param>

  <listener>
		<listener-class>
			org.springframework.web.context.ContextLoaderListener
		</listener-class>
  </listener>

  <!-- DispatcherServlet, Spring MVC的核心 -->
  <servlet>
		<servlet-name>mvc-dispatcher</servlet-name>
		<servlet-class> org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<!-- DispatcherServlet对应的上下文配置， 默认为/WEB-INF/$servlet-name$-servlet.xml
		 -->
		<init-param>
          <param-name>contextConfigLocation</param-name>
          <param-value>/WEB-INF/configs/spring/mvc-dispatcher-servlet.xml</param-value>
        </init-param>
		<load-on-startup>1</load-on-startup>
	</servlet>
	<servlet-mapping>
		<servlet-name>mvc-dispatcher</servlet-name>
	    <!-- mvc-dispatcher拦截所有的请求-->
		<url-pattern>/</url-pattern>
	</servlet-mapping>
</web-app>

```
继续对springmvc进行详细配置
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
	xmlns:mvc="http://www.springframework.org/schema/mvc"
	xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd">

	<!-- 本配置文件是工名为mvc-dispatcher的DispatcherServlet使用， 提供其相关的Spring MVC配置 -->

	<!-- 启用Spring基于annotation的DI, 使用户可以在Spring MVC中使用Spring的强大功能。 激活 @Required
		@Autowired,JSR 250's @PostConstruct, @PreDestroy and @Resource 等标注 -->
	<context:annotation-config />

	<!-- DispatcherServlet上下文， 只管理@Controller类型的bean， 忽略其他型的bean, 如@Service -->
	<context:component-scan base-package="com.imooc.mvcdemo">
		<context:include-filter type="annotation"
			expression="org.springframework.stereotype.Controller" />
	</context:component-scan>

	<!-- HandlerMapping, 无需配置， Spring MVC可以默认启动。 DefaultAnnotationHandlerMapping
		annotation-driven HandlerMapping -->

	<!-- 扩充了注解驱动，可以将请求参数绑定到控制器参数 -->
	<mvc:annotation-driven />

	<!-- 静态资源处理， css， js， imgs -->
	<mvc:resources mapping="/resources/**" location="/resources/" />


	<!-- 配置ViewResolver。 可以用多个ViewResolver。 使用order属性排序。 InternalResourceViewResolver放在最后。 -->
	<bean
		class="org.springframework.web.servlet.view.ContentNegotiatingViewResolver">
		<property name="order" value="1" />
		<property name="mediaTypes">
			<map>
				<entry key="json" value="application/json" />
				<entry key="xml" value="application/xml" />
				<entry key="htm" value="text/html" />
			</map>
		</property>

		<property name="defaultViews">
			<list>
				<!-- JSON View -->
				<bean
					class="org.springframework.web.servlet.view.json.MappingJackson2JsonView">
				</bean>
			</list>
		</property>
		<property name="ignoreAcceptHeader" value="true" />
	</bean>

	<bean
		class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<property name="viewClass"
			value="org.springframework.web.servlet.view.JstlView" />
		<property name="prefix" value="/WEB-INF/jsps/" />
		<property name="suffix" value=".jsp" />
	</bean>


	<!--200*1024*1024即200M resolveLazily属性启用是为了推迟文件解析，以便捕获文件大小异常 -->
	<bean id="multipartResolver"
		class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
		<property name="maxUploadSize" value="209715200" />
		<property name="defaultEncoding" value="UTF-8" />
		<property name="resolveLazily" value="true" />
	</bean>

</beans>
```

## 常用注解
### `@RequestMapping`
- value：指定请求的实际url
- method：指定请求的method类型， GET、POST、PUT、DELETE等
`@RequestMapping(value="/get/{bookid}",method={RequestMethod.GET,RequestMethod.POST})`

- params：指定request中必须包含某些参数值是，才让该方法处理。
`@RequestMapping(params="action=del")，请求参数包含“action=del”,如：http://localhost:8080/book?action=del`
```
@Controller
@RequestMapping("/owners/{ownerId}")
public class RelativePathUriTemplateController {

  @RequestMapping(value = "/pets/{petId}", method = RequestMethod.GET, params="myParam=myValue")
  public void findPet(@PathVariable String ownerId, @PathVariable String petId, Model model) {    
    // implementation omitted
  }
}
```
仅处理请求中包含了名为“myParam”，值为“myValue”的请求。

- headers：指定request中必须包含某些指定的header值，才能让该方法处理请求。
`@RequestMapping(value="/header/id", headers = "Accept=application/json")`：表示请求的URL必须为“/header/id 且请求头中必须有“Accept =application/json”参数即可匹配。

- consumes：指定处理请求的提交内容类型（Content-Type），例如application/json, text/html。
```
@Controller
@RequestMapping(value = "/pets", method = RequestMethod.POST, consumes="application/json")
public void addPet(@RequestBody Pet pet, Model model) {    
    // implementation omitted
}
```
方法仅处理request Content-Type为“application/json”类型的请求。
- produces: 指定返回的内容类型，仅当request请求头中的(Accept)类型中包含该指定类型才返回。

### `@RequestParam`
`@RequestParam`用于将请求参数区数据映射到功能处理方法的参数上。
比如`public String requestparam1(@RequestParam String username)` 请求中包含username参数（如/requestparam1?username=zhang），则自动传入。

`@RequestParam`有以下三个参数：
- value：参数名字，即入参的请求参数名字，如username表示请求的参数区中的名字为username的参数的值将传入；
- required：是否必须，默认是true，表示请求中一定要有相应的参数，否则将抛出异常；
- defaultValue：默认值，表示如果请求中没有同名参数时的默认值，设置该参数时，自动将required设为false。

`public String requestparam4(@RequestParam(value="username",required=false) String username)`
表示请求中可以没有名字为username的参数，如果没有默认为null，此处需要注意如下几点：
原子类型：必须有值，否则抛出异常，如果允许空值请使用包装类代替。
Boolean包装类型：默认Boolean.FALSE，其他引用类型默认为null。


如果请求参数类似于url?role=admin&rule=user，则实际roleList参数入参的数据为“admin,user”，即多个数据之间使用“，”分割；
我们应该使用如下方式来接收多个请求参数：
a）`public String requestparam7(@RequestParam(value="role") String[] roleList)`
b)`public String requestparam8(@RequestParam(value="list") List<String> list)  `

### `@PathVariable`
@PathVariable用于将请求URL中的模板变量映射到功能处理方法的参数上。
```
@RequestMapping(value="/users/{userId}/topics/{topicId}")
public String test(
       @PathVariable(value="userId") int userId,
       @PathVariable(value="topicId") int topicId)   
```
如请求的URL为“控制器URL/users/123/topics/456”，则自动将URL中模板变量{userId}和{topicId}绑定到通过`@PathVariable`注解的同名参数上，即入参后userId=123、topicId=456。

### `@RequestBody`
`@ResponseBody`作用：
该注解用于将Controller的方法返回的对象，通过适当的HttpMessageConverter转换为指定格式后，写入到Response对象的body数据区。
使用时机：
返回的数据不是html标签的页面，而是其他某种格式的数据时（如json、xml等）使用；

### `@ModelAttribute`
#### 注释方法
被@ModelAttribute注释的方法表示这个方法的目的是**增加一个或多个模型(model)属性**。
这个方法和被@RequestMapping注释的方法一样也支持@RequestParam参数，但是它不能直接被请求映射。
实际上，控制器中的@ModelAttribute方法是在**同一控制器中的@RequestMapping方法被调用之前调用的**。
#### 注释参数--数据绑定
`@ModelAttribute`注释方法的一个参数表示应从模型model中取得。
若在model中未找到，那么这个参数将先被实例化后加入到model中。若在model中找到，则请求参数名称和model属性字段若相匹配就会自动填充。这个机制对于表单提交数据绑定到对象属性上很有效。

当`@ModelAttribute`注解用于方法参数时，它有了双重功能，即“存/取”。
首先，它从模型中取出数据并赋予对应的参数，如果模型中尚不存在，则实例化一个，并存放于模型中；
其次，一旦模型中已存在此数据对象，接下来一个很重要的步骤便是将请求参数绑定到此对象上（请求参数名映射对象属性名）
这是Spring MVC提供的一个非常便利的机制--数据绑定。

当`@ModelAttribute`注解用于方法时，与其处于同一个处理类的所有请求方法执行前都会执行一次此方法，这可能并不是我们想要的
因此，我们使用更多的是将其应用在请求方法的参数上，而它的一部分功能与`@RequestParam`注解是一致的，只不过`@RequestParam`用于绑定单个参数值，
而`@ModelAttribute`注解可以绑定所有名称匹配的，此外它自动将绑定后的数据添加到模型中，无形中也给我们提供了便利，这也可能是它命名为ModelAttribute的原因。


http://www.cnblogs.com/xiaoxi/p/5718894.html

## redirect和forward

>forward 转发，如return "forward:/hello"; 浏览器的地址栏不会变，但是有视图返回来

forward方式相当于request.getRequestDispatcher().forward(request,response)，这种方式的外部特征是浏览器地址显示的路径是转发前的路径.工作方式是这样,forward 发生在服务器内部,在前一个控制器处理完毕后,直接进入下一个控制器处理, 并将最后的response发给浏览器. 这种方式的结果是:

A.转发前后是同一个request,后一个控制器可共享前一个控制器的参数与属性;
B.因为是同一个request,拦截器只会拦截前一个url,如果前一个url在映射时未配置到拦截器拦截，则拦截后一个url，即只拦截一次;
C.最后返回到浏览器后,因为地址栏显示的是转发前的url,所以刷新页面时会依次执行前后两个控制器.

>redirect 重定向，如return "redirect:/hello"; 浏览器的地址栏会变。

spring控制器最后返回一个ModelAndView(urlName),其中urNamel可以是一个视图名称,由视图解析器负责解析后将响应流写回客户端;也可以通过redirect/forward:url方式转到另一个控制器进行处理.

redirect方式相当于”response.sendRedirect()”.这种方式外部特征就是浏览器地址栏最后显示的路径是转发后的新的路径.工作方式是这样的, 服务器端会首先发一个response给浏览器, 然后浏览器收到这个response后再发一个requeset给服务器, 然后服务器发新的response给浏览器. 这时页面收到的request是一个新从浏览器发来的.这种方式的结果是:

A.在转发前后有两个不同的request对象,转发前后的两个控制器在request上的参数(request.getParameter())和request属性(request.getAttribute())不能共享;
B.如果转发前后的两个控制器都配置在spring 拦截器范围内,这样拦截器会拦截前后两个request,即会拦截两次;
C.最后返回到浏览器后,因为地址栏显示的是转发后的url,所以刷新页面时只会执行后面的url映射的控制器.

## springmvc与struts2的区别
1、Struts2是类级别的拦截， 一个类对应一个request上下文，SpringMVC是方法级别的拦截，一个方法对应一个request上下文，而方法同时又跟一个url对应,所以说从架构本身上SpringMVC就容易实现restful url,而struts2的架构实现起来要费劲，因为Struts2中Action的一个方法可以对应一个url，而其类属性却被所有方法共享，这也就无法用注解或其他方式标识其所属方法了。
2、由上边原因，SpringMVC的方法之间基本上独立的，独享request response数据，请求数据通过参数获取，处理结果通过ModelMap交回给框架，方法之间不共享变量，而Struts2搞的就比较乱，虽然方法之间也是独立的，但其所有Action变量是共享的，这不会影响程序运行，却给我们编码 读程序时带来麻烦，每次来了请求就创建一个Action，一个Action对象对应一个request上下文。
3、由于Struts2需要针对每个request进行封装，把request，session等servlet生命周期的变量封装成一个一个Map，供给每个Action使用，并保证线程安全，所以在原则上，是比较耗费内存的。
4、 拦截器实现机制上，Struts2有以自己的interceptor机制，SpringMVC用的是独立的AOP方式，这样导致Struts2的配置文件量还是比SpringMVC大。
5、SpringMVC的入口是servlet，而Struts2是filter（这里要指出，filter和servlet是不同的。以前认为filter是servlet的一种特殊），这就导致了二者的机制不同，这里就牵涉到servlet和filter的区别了。
6、SpringMVC集成了Ajax，使用非常方便，只需一个注解@ResponseBody就可以实现，然后直接返回响应文本即可，而Struts2拦截器集成了Ajax，在Action中处理时一般必须安装插件或者自己写代码集成进去，使用起来也相对不方便。
http://blog.csdn.net/chenleixing/article/details/44570681
http://blog.csdn.net/gstormspire/article/details/8239182


## 线程安全问题
对于使用过SpringMVC和Struts2的人来说，大家都知道SpringMVC是基于方法的拦截，而Struts2是基于类的拦截。

对于Struts2来说，因为每次处理一个请求，struts就会实例化一个对象；这样就不会有线程安全的问题了;
而Spring的controller默认是Singleton的，这意味着每一个request过来，系统都会用原有的instance去处理，这样导致两个结果：
**一是我们不用每次创建Controller，二是减少了对象创建和垃圾收集的时间;**
由于只有一个Controller的instance，当多个线程调用它的时候，它里面的instance变量就不是线程安全的了，会发生窜数据的问题。

当然大多数情况下，我们根本不需要考虑线程安全的问题，比如dao,service等，除非在bean中声明了实例变量。因此，我们在使用spring mvc 的contrller时，应避免在controller中定义实例变量。
如：
```java
public class Controller extends AbstractCommandController {    
    protected Company company;  
    protected ModelAndView handle(HttpServletRequest request,HttpServletResponse response,Object command,BindException errors) throws Exception {  
        company = ................;           
     }             
 }   
```

解决方案：
有几种解决方法：
1、在Controller中使用ThreadLocal变量
2、在spring配置文件Controller中声明 scope="prototype"，每次都创建新的controller
所在在使用spring开发web 时要注意，默认Controller、Dao、Service都是单例的。

http://blog.csdn.net/hejingyuan6/article/details/50363647

如果在spring中使用多线程
http://blog.csdn.net/hejingyuan6/article/details/51498401

# ibatis
主要完成两件事情:
根据jdbc完成数据库的连接
通过反射打通java对象与数据库交互

![ibatis的主要方法](http://7xilc8.com1.z0.glb.clouddn.com/ibatis.png)
`SqlMapClient`接口主要定义了客户端的操作行为比如insert、update、select和delete
`SqlMapSession`主要定义了客户端的当前执行环境，可以共享也可以自己创建
但一个用户持有了`SqlMapClientImpl`就可以使用ibatis工作了


iBATIS 框架的一个重要组成部分就是其 SqlMap 配置文件，SqlMap 配置文件的核心是 Statement 语句包括 CIUD。 iBATIS 通过解析 SqlMap 配置文件得到所有的 Statement 执行语句，同时会形成 ParameterMap、ResultMap 两个对象用于处理参数和经过解析后交给数据库处理的 Sql 对象。这样除去数据库的连接，一条 SQL 的执行条件已经具备了。

数据的映射大体的过程是这样的：根据 Statement 中定义的 SQL 语句，解析出其中的参数，按照其出现的顺序保存在 Map 集合中，并按照 Statement 中定义的 ParameterMap 对象类型解析出参数的 Java 数据类型。并根据其数据类型构建 TypeHandler 对象，参数值的复制是通过 DataExchange 对象完成的。

http://blog.csdn.net/column/details/mybatis-principle.html

https://www.ibm.com/developerworks/cn/java/j-lo-ibatis-principle/index.html


## mybatis初始化过程
```
String resource = "mybatis-config.xml";  
InputStream inputStream = Resources.getResourceAsStream(resource);  
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);  
SqlSession sqlSession = sqlSessionFactory.openSession();  
List list = sqlSession.selectList("com.foo.bean.BlogMapper.queryAllBlogInfo");  
```
上述语句的作用是执行com.foo.bean.BlogMapper.queryAllBlogInfo 定义的SQL语句，返回一个List结果集。
总的来说，上述代码经历了**mybatis初始化 -->创建SqlSession -->执行SQL语句** 返回结果三个过程
`new SqlSessionFactoryBuilder().build(inputStream)`这句话是SqlSessionFactoryBuilder根据传入的数据流生成Configuration对象，然后根据Configuration对象创建默认的SqlSessionFactory实例。

http://blog.csdn.net/luanlouis/article/details/37744073

## Mybatis数据源与连接池
MyBatis把数据源DataSource分为三种：
- UNPOOLED    不使用连接池的数据源
- POOLED      使用连接池的数据源
- JNDI            使用JNDI实现的数据源

### 创建DataSource
MyBatis数据源DataSource对象的创建发生在MyBatis初始化的过程中。
在mybatis的XML配置文件中，使用<dataSource>元素来配置数据源

### 创建connection
当我们需要创建SqlSession对象并需要执行SQL语句时，这时候MyBatis才会去调用dataSource对象来创建java.sql.Connection对象。也就是说，java.sql.Connection对象的创建一直延迟到执行SQL语句的时候。


- 不使用连接池
使用UnpooledDataSource的getConnection(),每调用一次就会产生一个新的Connection实例对象。

- 使用连接池
使用PooledDataSource的getConnection()方法来返回Connection对象。
PooledDataSource将java.sql.Connection对象包裹成PooledConnection对象放到了PoolState类型的容器中维护。
MyBatis将连接池中的PooledConnection分为两种状态： `空闲状态（idle）`和`活动状态(active)`，这两种状态的PooledConnection对象分别被存储到PoolState容器内的idleConnections和activeConnections两个List集合中：

idleConnections:空闲(idle)状态PooledConnection对象被放置到此集合中，表示当前闲置的没有被使用的PooledConnection集合，调用PooledDataSource的getConnection()方法时，会优先从此集合中取PooledConnection对象。当用完一个java.sql.Connection对象时，MyBatis会将其包裹成PooledConnection对象放到此集合中。

activeConnections:活动(active)状态的PooledConnection对象被放置到名为activeConnections的ArrayList中，表示当前正在被使用的PooledConnection集合，调用PooledDataSource的getConnection()方法时，会优先从idleConnections集合中取PooledConnection对象,如果没有，则看此集合是否已满，如果未满，PooledDataSource会创建出一个PooledConnection，添加到此集合中，并返回。

### 用完connection
使用代理模式，为真正的Connection对象创建一个代理对象，代理对象所有的方法都是调用相应的真正Connection对象的方法实现。当代理对象执行close()方法时，要特殊处理，不调用真正Connection对象的close()方法，而是将Connection对象添加到连接池中。

http://blog.csdn.net/luanlouis/article/details/37671851

## 缓存
### 一级缓存
每当我们使用MyBatis开启一次和数据库的会话，MyBatis会创建出一个SqlSession对象表示一次数据库会话。
在对数据库的一次会话中，我们有可能会反复地执行完全相同的查询语句，如果不采取一些措施的话，每一次查询都会查询一次数据库,而我们在极短的时间内做了完全相同的查询，那么它们的结果极有可能完全相同，由于查询一次数据库的代价很大，这有可能造成很大的资源浪费。
为了解决这一问题，减少资源的浪费，MyBatis会在表示会话的SqlSession对象中建立一个简单的缓存，将每次查询到的结果结果缓存起来，当下次查询的时候，如果判断先前有个完全一样的查询，会直接从缓存中直接将结果取出，返回给用户，不需要再进行一次数据库查询了。

MyBatis会在一次会话的表示----一个SqlSession对象中创建一个本地缓存(local cache)，对于每一次查询，都会尝试根据查询的条件去本地缓存中查找是否在缓存中，如果在缓存中，就直接从缓存中取出，然后返回给用户；否则，从数据库读取数据，将查询结果存入缓存并返回给用户。
![mybatis一级缓存](http://7xilc8.com1.z0.glb.clouddn.com/mb1.png)

实际上, SqlSession只是一个MyBatis对外的接口，SqlSession将它的工作交给了Executor执行器这个角色来完成，负责完成对数据库的各种操作。
当创建了一个SqlSession对象时，MyBatis会为这个SqlSession对象创建一个新的Executor执行器，而缓存信息就被维护在这个Executor执行器中，MyBatis将缓存和对缓存相关的操作封装成了Cache接口中。


#### 一级缓存的生命周期
a. MyBatis在开启一个数据库会话时，会创建一个新的SqlSession对象，SqlSession对象中会有一个新的Executor对象，Executor对象中持有一个新的PerpetualCache对象；当会话结束时，SqlSession对象及其内部的Executor对象还有PerpetualCache对象也一并释放掉。
b. 如果SqlSession调用了close()方法，会释放掉一级缓存PerpetualCache对象，一级缓存将不可用；
c. 如果SqlSession调用了clearCache()，会清空PerpetualCache对象中的数据，但是该对象仍可使用；
d.SqlSession中执行了任何一个update操作(update()、delete()、insert()) ，都会清空PerpetualCache对象的数据，但是该对象可以继续使用；

#### Cache原理
Cache最核心的实现其实就是一个Map，将本次查询使用的特征值作为key，将查询结果作为value存储到Map中。
怎样来确定一次查询的特征值？换句话说就是：怎样判断某两次查询是完全相同的查询？也可以这样说：如何确定Cache中的key值？
MyBatis认为，对于两次查询，如果以下条件都完全一样，那么就认为它们是完全相同的两次查询：
1. 传入的 statementId
2. 查询时要求的结果集中的结果范围 （结果的范围通过rowBounds.offset和rowBounds.limit表示）；
3. 这次查询所产生的最终要传递给JDBC Java.sql.Preparedstatement的Sql语句字符串（boundSql.getSql() ）
4. 传递给java.sql.Statement要设置的参数值
综上所述,CacheKey由以下条件决定：statementId  + rowBounds  + 传递给JDBC的SQL  + 传递给JDBC的参数值

#### 总结
1.MyBatis对会话（Session）级别的一级缓存设计的比较简单，就简单地使用了HashMap来维护，并没有对HashMap的容量和大小进行限制。
读者有可能就觉得不妥了：如果我一直使用某一个SqlSession对象查询数据，这样会不会导致HashMap太大，而导致 java.lang.OutOfMemoryError错误啊？ 读者这么考虑也不无道理，不过MyBatis的确是这样设计的。
MyBatis这样设计也有它自己的理由：
a.  一般而言SqlSession的生存时间很短。一般情况下使用一个SqlSession对象执行的操作不会太多，执行完就会消亡；
b.  对于某一个SqlSession对象而言，只要执行update操作（update、insert、delete），都会将这个SqlSession对象中对应的一级缓存清空掉，所以一般情况下不会出现缓存过大，影响JVM内存空间的问题；
c.  可以手动地释放掉SqlSession对象中的缓存。
2.  一级缓存是一个粗粒度的缓存，没有更新缓存和缓存过期的概念
MyBatis的一级缓存就是使用了简单的HashMap，MyBatis只负责将查询数据库的结果存储到缓存中去， 不会去判断缓存存放的时间是否过长、是否过期，因此也就没有对缓存的结果进行更新这一说了。

根据一级缓存的特性，在使用的过程中，我认为应该注意：
1、对于数据变化频率很大，并且需要高时效准确性的数据要求，我们使用SqlSession查询的时候，要控制好SqlSession的生存时间，SqlSession的生存时间越长，它其中缓存的数据有可能就越旧，从而造成和真实数据库的误差；同时对于这种情况，用户也可以手动地适时清空SqlSession中的缓存；
2、对于只执行、并且频繁执行大范围的select操作的SqlSession对象，SqlSession对象的生存时间不应过长。


### 二级缓存
MyBatis并不是简单地对整个Application就只有一个Cache缓存对象，它将缓存划分的更细，即是Mapper级别的，即每一个Mapper都可以拥有一个Cache对象，具体如下：
a.为每一个Mapper分配一个Cache缓存对象（使用<cache>节点配置）；
b.多个Mapper共用一个Cache缓存对象（使用<cache-ref>节点配置）；

![mybatis二级缓存](http://7xilc8.com1.z0.glb.clouddn.com/mb2.png)

#### 具备的条件
MyBatis对二级缓存的支持粒度很细，它会指定某一条查询语句是否使用二级缓存。
虽然在Mapper中配置了`<cache>`,并且为此Mapper分配了Cache对象，这并不表示我们使用Mapper中定义的查询语句查到的结果都会放置到Cache对象之中，我们必须指定Mapper中的某条选择语句是否支持缓存，即如下所示，在`<select>` 节点中配置useCache="true"，Mapper才会对此Select的查询支持缓存特性，否则，不会对此Select查询，不会经过Cache缓存。如下所示，Select语句配置了useCache="true"，则表明这条Select语句的查询会使用二级缓存。
`<select id="selectByMinSalary" resultMap="BaseResultMap" parameterType="java.util.Map" useCache="true">  `

总之，要想使某条Select查询支持二级缓存，你需要保证：
1.  MyBatis支持二级缓存的总开关：全局配置变量参数   cacheEnabled=true
2. 该select语句所在的Mapper，配置了<cache> 或<cached-ref>节点，并且有效
3. 该select语句的参数 useCache=true


MyBatis查询数据的顺序是： 二级缓存    ———> 一级缓存——> 数据库
http://blog.csdn.net/luanlouis/article/details/41408341

## `#`与`$`区别
一般如果可以使用`#`就不要使用`$`符号。
1）使用`#`入参，myBatis会生成PrepareStatement并且可以安全地设置参数（`=?`）的值。因为sql语句已经预编译好了，传入参数的时候，不会重新生产sql语句。安全性高。
2）用`$`可以会有sql注入的问题：例如，`select * from emp where ename = '用户名'`，如果使用`$`入参，用户名被传入例如`smith or 1 = 1`，那无论ename是否匹配都能查到结果。
3）在特定场景下，例如如果在使用诸如`order by '{param}'`，这时候就可以使用`$`.

http://www.mybatis.org/mybatis-3/sqlmap-xml.html#String_Substitution

# velocity
Velocity 是一个基于 Java 的模板引擎框架，提供的模板语言可以使用在 Java 中定义的对象和变量上。
Velocity 是 Apache 基金会的项目，开发的目标是分离 MVC 模式中的持久化层和业务层。但是在实际应用过程中，Velocity 不仅仅被用在了 MVC 的架构中，还可以被用在以下一些场景中。
1.Web 应用：开发者在不使用 JSP 的情况下，可以用 Velocity 让 HTML 具有动态内容的特性。
2.源代码生成：Velocity 可以被用来生成 Java 代码、SQL 或者 PostScript。有很多开源和商业开发的软件是使用 Velocity 来开发的。
3.自动 Email：很多软件的用户注册、密码提醒或者报表都是使用 Velocity 来自动生成的。使用 Velocity 可以在文本文件里面生成邮件内容，而不是在 Java 代码中拼接字符串。
4.转换 xml：Velocity 提供一个叫 Anakia 的 ant 任务，可以读取 XML 文件并让它能够被 Velocity 模板读取。一个比较普遍的应用是将 xdoc 文档转换成带样式的 HTML 文件。
## 过程
![velocity过程](http://7xilc8.com1.z0.glb.clouddn.com/velocity.png)

Velocity模板引擎处理机制分为五个基本步骤：
a.引擎初始化，通过设置的引擎属性初始化引擎，包括国际化支持，ResourceLoader设置，字符编码等。
b.获取并解析模板文件，首先通过资源加载器（ResourceLoader）将模板文件加载到内存（转化为InputStream），然后通过AST（Abstract Syntax Tree）解析器将InputStream解析为一个AST。
c.创建一个Context
d.将模板渲染所需的参数放入context
e.执行模板渲染，产生输出流。渲染过程中通过遍历该模板对应的AST，调用相应节点的处理器执行渲染。


```java
public class VelocityMergeTest {
    public static void main(String[] args) {
          VelocityEngine ve = new VelocityEngine();
          ve.setProperty(Velocity.RESOURCE_LOADER, "class");
          //设置引擎的资源加载使用ClasspathResourceLoader。
          ve.setProperty("class.resource.loader.class",
          "org.apache.velocity.runtime.resource.loader.ClasspathResourceLoader");
          try {
            //引擎初始化
            ve.init();
            //加载解析模板
            Template tp = ve.getTemplate("template.vm");
            //创建context
            Context context = new VelocityContext();
            //设置Context中参数值
            //创建一个context，并在其中放入一个foo=VV的参数。
            context.put("foo", "VV");
            StringWriter writer = new StringWriter();
            //执行渲染
            tp.merge(context, writer);
            System.out.println(writer.toString());
          } catch (Exception e) {
          }
    }
}
```
- 加载解析模板
当执行ve.getTemplate("template.vm")时，首先通过ResourceLoader将tempalte加载为InputStream，然后通过Parser生成如下Token集合：
{[<html> <body> Hello], [$foo], [world! </body> </html> ]}，
可以发现velocity根本不关系模板最终要渲染出来的是html还是什么的其他的东西，也就以为这所有的html标签对velocity来讲都是纯文本。
最终构建的AST如下

![velocity过程](http://7xilc8.com1.z0.glb.clouddn.com/AST.png)
根节点下有三个子节点：

[<html> <body> Hello]对应的ASTText节点；
[$foo]对应的ASTReference节点；
[world! </body> </html> ]对应的ASTText节点
Velocity引擎在这里有个优化策略，可以针对生成的语法树进行cache。

- 执行渲染

当执行tp.merge(context, writer);时，模板遍历其对应的AST树，执行每个节点的渲染过程。
如ASTText节点只是简单的将文本写入writer。ASTReference节点需要从context中获取引用的参数foo的值VV，将$foo替换，并写入到writer中。
Velocity的AST中有多种节点，如ASTIdentitor等，有些需要反射机制处理。当整个AST遍历结束，也就意味着模板渲染结束，渲染的结果位于writer流中。

## 关于AST解析
Velocity作为模板语言，其核心在与模板文件的解析，构建AST。Velocity的解析器是通过JavaCC构建的，JavaCC（Java Complier Complier）是一个用于生成解析器的工具，可以根据语法定义（.jj文件）生成用于校验一份文本是否符合该语法定义的java代码。JJTree是JavaCC中提供的一种根据语法定义（.jjt文件）生成构建符合该语法定义的文本的语法树的java代码的工具。

## 常用优化技巧

Velocity渲染模板是先把模板解析成一棵语法树，然后去遍历这棵树分别渲染每个节点，知道了它的工作原理，我们就可以根据它的工作机制来优化渲染的速度。
既然是遍历这棵树来渲染节点的，而且是顺序遍历的，那么很容易想到有两种办法来优化渲染：

- 减少树的总节点数量。
```
#set($one=-1)
#set($pageid="$!page.id")
#set($page=$one+$pageid)
```
这段代码实际上只是要计算一个值，但是由于不熟悉Velocity的一些语法，写得很麻烦，其实只要一个表达式就好了，如下：
`#set($page=-1+$!page.id)`
这样可以减少很多语法节点。
- 减少渲染耗时的节点数量。
Velocity的方法调用是通过反射执行的，显然反射执行方法是耗时的，那么又如何减少反射执行的方法呢？
这个改进就如同Java中一样，可以增加一些中间变量来保存中间值，而减少反射方法的调用。
如在一个模板中要多次调用到$person.name，那么可以通过#set创建一个变量$name来保存$person.name这个反射方法的执行结果。
如#set($name=$person.name)，这样虽然增加了一个#set节点，但是如果能减少多次反射调用仍然是很值得的。
另外，Velocity本身提供了一个#macro语法，它类似于定义一个方法，然后可以调用这个方法，但在没有必要时尽量少用这种语法节点，这些语法节点比较耗时。还有一些大数计算等，最好定义在Java中，通过调用Java中的方法可以加快Velocity的执行效率。


其他：


- 改变Velocity的解释执行，变为编译执行。
- 方法调用的无反射优化
- 字符输出改成字节输出
静态字符串直接是`out.write(_S0)`，这里的_S0是一个字节数组，而vm模板中是字符串，将字符串转成字节数组是在这个模板类初始化时完成的。字符的编码是非常耗时的，如果我们将静态字符串提前编码好，那么在最终写Socket流时就会省去这个编码时间，从而提高执行效率。从实际的测试来看，这对提升性能很有帮助。另外，从代码中还可以发现，如果是变量输出，调用的是`out.write(_EVTCK(context,"$str", context.get("str")))`，而_EVTCK方法在输出变量之前检查是否有事件需要调用，如XSS安全检查、为空检查等。
- 去掉页面输出中多余的非中文空格。我们知道，页面的HTML输出中多余的空格是不会在HTML的展示时有作用的，多个连续的空格最终都只会显示一个空格的间距，除非你使用“ ”表示空格。虽然多余的空格并不能影响HTML的页面展示样式，但是服务端页面渲染和网络数据传输这些空格和其他字符没有区别，同样要做处理，这样的话，这些空格就会造成时间和空间维度上的浪费，所以完全可以将多个连续的空格合并成一个，从而既减少了字符又不会影响页面展示。
- 压缩TAB和换行。同样的道理，还可以将TAB字符合并成一个，以及将多余的换行也合并一下，也能减少不少字符。
- 合并相同的数据。在模板中有很多相同数据在循环中重复输出，如类目、商品、菜单等，可以将相同的重复内容提取出来合并在CSS中或者用JS来输出。
异步渲染。将一些静态内容抽取出来改成异步渲染，只在用户确实需要时再向服务器去请求，也能够减少很多不必要的数据传输。


http://www.cnblogs.com/wade-luffy/p/5996848.html


## Velocity与JSP

JSP文件渲染其实和Velocity的渲染机制很不一样，JSP文件实际上执行的是JSP对应的Java类，简单地说就是将JSP的HTML转化成out.write输出，而JSP中的Java代码直接复制到翻译后的Java类中。
最终执行的是翻译后的Java类，而Velocity是按照语法规则解析成一棵语法树，然后执行这棵语法树来渲染出结果。

所以它们有如下这些区别:

执行方式不一样：JSP是编译执行，而Velocity是解释执行。如果JSP文件被修改了，那么对应的Java类也会被重新编译，而Velocity却不需要，只是会重新生成一棵语法树。

执行效率不同：从两者的执行方式不同可以看出，它们的执行效率不一样，从理论上来说，编译执行的效率明显好于解释执行，一个很明显的例子在JSP中方法调用是直接执行的，而Velocity的方法调用是反射执行的，JSP的效率会明显好于Velocity。当然如果JSP中有语法JSTL，语法标签的执行要看该标签的实现复杂度。
需要的环境支持不一样：JSP的执行必须要有Servlet的运行环境，也就是需要ServletContext、HttpServletRequest和HttpServletResponse类。而要渲染Velocity完全不需要其他环境类的支持，直接给定Velocity模板就可以渲染出结果。所以Velocity不只应用在Servlet环境中。
