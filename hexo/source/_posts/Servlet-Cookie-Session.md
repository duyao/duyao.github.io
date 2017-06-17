title: Servlet, Cookie & Session
toc: true
date: 2017-03-19 16:06:11
tags:
categories:
---
# Servlet
## 生命周期
Servlet 生命周期：Servlet 加载--->实例化--->服务--->销毁。

init（）：在Servlet的生命周期中，仅执行一次init()方法。它是在服务器装入Servlet时执行的，负责初始化Servlet对象。可以配置服务器，以在启动服务器或客户机首次访问Servlet时装入Servlet。无论有多少客户机访问Servlet，都不会重复执行init（）。
service（）：它是Servlet的核心，负责响应客户的请求。每当一个客户请求一个HttpServlet对象，该对象的Service()方法就要调用，而且传递给这个方法一个“请求”（ServletRequest）对象和一个“响应”（ServletResponse）对象作为参数。在HttpServlet中已存在Service()方法。默认的服务功能是调用与HTTP请求的方法相应的do功能。
destroy（）： 仅执行一次，在服务器端停止且卸载Servlet时执行该方法。当Servlet对象退出生命周期时，负责释放占用的资源。一个Servlet在运行service()方法时可能会产生其他的线程，因此需要确认在调用destroy()方法时，这些线程已经终止或完成。

## Tomcat 与 Servlet 的工作过程
一个web应用对应一个context容器
Context容器中可以解析web.xml文件，来完成初始化工作。


### 创建Servlet对象的时机
Servlet容器启动时：读取web.xml配置文件中的信息，构造指定的Servlet对象，创建ServletConfig对象，同时将ServletConfig对象作为参数来调用Servlet对象的init方法。
ServletConfig代表当前Servlet在web.xml中的配置信息。
在Servlet接口中init方法参数就是ServletConfig，在Servlet创建出来时，init方法立即被容器调用，由容器传入ServletConfig对象。
在GenericServlet中，实现了这个方法，将ServletConfig设置成了类的成员变量，并提供了getServletConfig方法，获取该对象。
我们的Servlet都间接继承子GenericServlet，所以可以在自己的Servlet中直接调用getServletConfig方法获取这个对象。

在Servlet容器启动后：客户首次向Servlet发出请求，Servlet容器会判断内存中是否存在指定的Servlet对象，如果没有则创建它，然后根据客户的请求创建HttpRequest、HttpResponse对象，从而调用Servlet 对象的service方法。

Servlet Servlet容器在启动时自动创建Servlet，这是由在`web.xml`文件中为Servlet设置的<load-on-startup>属性决定的。
从中我们也能看到同一个类型的Servlet对象在Servlet容器中以单例的形式存在。

具体过程：
1.Web Client 向Servlet容器（Tomcat）发出Http请求
2.Servlet容器接收Web Client的请求
3.Servlet容器创建一个HttpRequest对象，将Web Client请求的信息封装到这个对象中。
4.Servlet容器创建一个HttpResponse对象
5.Servlet容器调用HttpServlet对象的service方法，把HttpRequest对象与HttpResponse对象作为参数传给 HttpServlet 对象。
6.HttpServlet调用HttpRequest对象的有关方法，获取Http请求信息。
7.HttpServlet调用HttpResponse对象的有关方法，生成响应数据。
8.Servlet容器把HttpServlet的响应结果传给Web Client。





Servlet工作原理：

1、首先简单解释一下Servlet接收和响应客户请求的过程，首先客户发送一个请求，Servlet是调用service()方法对请求进行响应的，通过源代码可见，service()方法中对请求的方式进行了匹配，选择调用doGet,doPost等这些方法，然后再进入对应的方法中调用逻辑层的方法，实现对客户的响应。在Servlet接口和GenericServlet中是没有doGet（）、doPost（）等等这些方法的，HttpServlet中定义了这些方法，但是都是返回error信息，所以，我们每次定义一个Servlet的时候，都必须实现doGet或doPost等这些方法。

2、每一个自定义的Servlet都必须实现Servlet的接口，Servlet接口中定义了五个方法，其中比较重要的三个方法涉及到Servlet的生命周期，分别是上文提到的init(),service(),destroy()方法。GenericServlet是一个通用的，不特定于任何协议的Servlet,它实现了Servlet接口。而HttpServlet继承于GenericServlet，因此HttpServlet也实现了Servlet接口。所以我们定义Servlet的时候只需要继承HttpServlet即可。

3、Servlet接口和GenericServlet是不特定于任何协议的，而HttpServlet是特定于HTTP协议的类，所以HttpServlet中实现了service()方法，并将请求ServletRequest、ServletResponse 强转为HttpRequest 和 HttpResponse。

# Filter
Filter是一个可以复用的代码片段，可以用来转换HTTP请求、响应和头信息。
Filter不像Servlet，它不能产生一个请求或者响应，它只是修改对某一资源的请求，或者修改从某一的响应。
Servlet中的过滤器Filter是实现了javax.servlet.Filter接口的服务器端程序，主要的用途是过滤字符编码、做一些业务逻辑判断等。

过滤器分为request(默认)、forward、error、include、async

servlet3.0中添加webfilter通过注解方式配置类过滤器
## 工作原理
只要你在web.xml文件配置好要拦截的客户端请求，它都会帮你拦截到请求，此时你就可以对请求或响应(Request、Response)统一设置编码，简化操作；同时还可进行逻辑判断，如用户是否已经登陆、有没有权限访问该页面等等工作。
它是随你的web应用启动而启动的，只初始化一次，以后就可以拦截相关请求，只有当你的web应用停止或重新部署的时候才销毁。
Filter可认为是Servlet的一种“变种”，它主要用于对用户请求进行预处理，也可以对HttpServletResponse进行后处理，是个典型的处理链。

它与Servlet的区别在于：它不能直接向用户生成响应。
完整的流程是：Filter对用户请求进行预处理，接着将请求交给Servlet进行处理并生成响应，最后Filter再对服务器响应进行后处理。

Filter有如下几个用处:
在HttpServletRequest到达Servlet之前，拦截客户的HttpServletRequest。
根据需要检查HttpServletRequest，也可以修改HttpServletRequest头和数据。
在HttpServletResponse到达客户端之前，拦截HttpServletResponse。
根据需要检查HttpServletResponse，也可以修改HttpServletResponse头和数据。

Filter有如下几个种类:
request(默认)、forward、error、include、async

创建一个Filter只需两个步骤：
建Filter处理类；
web.xml文件中配置Filter。**web服务器根据Filter在web.xml文件中的注册顺序，决定先调用哪个Filter**
```
<filter-name>用于为过滤器指定一个名字，该元素的内容不能为空。
<filter-class>元素用于指定过滤器的完整的限定类名。
<init-param>元素用于为过滤器指定初始化参数，它的子元素<param-name>指定参数的名字，<param-value>指定参数的值。
在过滤器中，可以使用FilterConfig接口对象来访问初始化参数。

<filter-mapping>元素用于设置一个 Filter 所负责拦截的资源。一个Filter拦截的资源可通过两种方式来指定：Servlet 名称和资源访问的请求路径
<filter-name>子元素用于设置filter的注册名称。该值必须是在<filter>元素中声明过的过滤器的名字
<url-pattern>设置 filter 所拦截的请求路径(过滤器关联的URL样式)
<servlet-name>指定过滤器所拦截的Servlet名称。
<dispatcher>指定过滤器所拦截的资源被 Servlet 容器调用的方式，可以是REQUEST,INCLUDE,FORWARD和ERROR之一，默认REQUEST。用户可以设置多个<dispatcher> 子元素用来指定 Filter 对资源的多种调用方式进行拦截。

<dispatcher> 子元素可以设置的值及其意义：
REQUEST：当用户直接访问页面时，Web容器将会调用过滤器。如果目标资源是通过RequestDispatcher的include()或forward()方法访问时，那么该过滤器就不会被调用。
INCLUDE：如果目标资源是通过RequestDispatcher的include()方法访问时，那么该过滤器将被调用。除此之外，该过滤器不会被调用。
FORWARD：如果目标资源是通过RequestDispatcher的forward()方法访问时，那么该过滤器将被调用，除此之外，该过滤器不会被调用。
ERROR：如果目标资源是通过声明式异常处理机制调用时，那么该过滤器将被调用。除此之外，过滤器不会被调用。
```
```java
public class FilterDemo2 implements Filter{

   @Override
   public void init(FilterConfig filterConfig) throws ServletException {
       // TODO Auto-generated method stub
   }
   @Override
   public void doFilter(ServletRequest request, ServletResponse response,
         FilterChain chain) throws IOException, ServletException {
       // TODO Auto-generated method stub
       System.out.println("我是FilterDemo2，客户端向Servlet发送的请求被我拦截到了");
       //对请求放行，进入Servlet
       chain.doFilter(request, response);
       System.out.println("我是FilterDemo2，Servlet向客户端发送的响应被我拦截到了");
   }
   @Override
   public void destroy() {
       // TODO Auto-generated method stub
   }
}
```

http://www.tuicool.com/articles/RFfMBr7
http://www.jianshu.com/p/ee5bdbb0f37f
# listener：监听器
listener主要用来监听只用。通过listener可以监听web服务器中某一个执行动作，并根据其要求作出相应的响应。
通俗的语言说就是在application，session，request三个对象创建消亡或者往其中添加修改删除属性时自动执行代码的功能组件。

比如spring 的总监听器会在服务器启动的时候实例化我们配置的bean对象 、 hibernate 的 session 的监听器会监听session的活动和生命周期，负责创建，关闭session等活动。

Servlet的监听器Listener，它是实现了javax.servlet.ServletContextListener 接口的服务器端程序，它也是随web应用的启动而启动，只初始化一次，随web应用的停止而销毁。
主要作用是： 做一些初始化的内容添加工作、设置一些基本的内容、比如一些参数或者是一些固定的对象等等。

# Interceptor
在面向切面编程的，就是在你的service或者一个方法，前调用一个方法，或者在方法后调用一个方法，是基于JAVA的反射机制。
比如动态代理就是拦截器的简单实现，在你调用方法前打印出字符串（或者做其它业务逻辑的操作），也可以在你调用方法后打印出字符串，甚至在你抛出异常的时候做业务逻辑的操作。
## 使用方法
实现`HandlerInterceptor`接口，重写`preHandle`、`postHandle`、`afterCompletion`方法

`preHandle`：预处理回调方法，实现处理器的预处理（如登录检查），第三个参数为响应的处理器（如我们上一章的Controller实现）；返回值：true表示继续流程（如调用下一个拦截器或处理器）；false表示流程中断（如登录检查失败），不会继续调用其他的拦截器或处理器，此时我们需要通过response来产生响应；
`postHandle`：后处理回调方法，**实现处理器的后处理（但在渲染视图之前）**，此时我们可以通过modelAndView（模型和视图对象）对模型数据进行处理或对视图进行处理，modelAndView也可能为null。
`afterCompletion`：整个请求处理完毕回调方法，即在**视图渲染完毕时回调**，如性能监控中我们可以在此记录结束时间并输出消耗时间，还可以进行一些资源清理，类似于try-catch-finally中的finally，但**仅调用处理器执行链中preHandle返回true的拦截器的afterCompletion**。
```java
public class LoginRequiredInterceptor implements HandlerInterceptor {

    @Autowired
    private HostHolder hostHolder;

    @Override
    public boolean preHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o) throws Exception {
        if (hostHolder.getUser() == null) {
            httpServletResponse.sendRedirect("/reglogin?next=" + httpServletRequest.getRequestURI());
            return false;
        }
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {
    }

    @Override
    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {
    }
}
```
# 区别
servlet、filter、listener是配置到web.xml中
web.xml 的加载顺序是：context-param -> listener -> filter -> servlet
interceptor不配置到web.xml中，struts的拦截器配置到struts.xml中。spring的拦截器配置到spring.xml中。


过滤器和拦截器的区别：
　　①拦截器是基于Java的反射机制的，而过滤器是基于函数回调。
　　②拦截器不依赖与servlet容器，过滤器依赖与servlet容器。
　　③拦截器只能对action请求起作用，而过滤器则可以对几乎所有的请求起作用。
　　④拦截器可以访问action上下文、值栈里的对象，而过滤器不能访问。
　　⑤在action的生命周期中，拦截器可以多次被调用，而过滤器只能在容器初始化时被调用一次。
　　⑥拦截器可以获取IOC容器中的各个bean，而过滤器就不行，这点很重要，在拦截器里注入一个service，可以调用业务逻辑。

![过滤器和拦截器的区别](http://7xilc8.com1.z0.glb.clouddn.com/servletcompare.jpg)
spring优先使用拦截器，其功能更强大
http://blog.csdn.net/chenleixing/article/details/44573495
# jsp

JSP技术使用Java编程语言编写类XML的tags和scriptlets，来封装产生动态网页的处理逻辑。
网页还能通过tags和scriptlets访问存在于服务端的资源的应用逻辑。
JSP将网页逻辑与网页设计的显示分离，支持可重用的基于组件的设计，使基于Web的应用程序的开发变得迅速和容易。
JSP(JavaServer Pages)是一种动态页面技术，它的主要目的是将表示逻辑从Servlet中分离出来。

## 与servlet的区别
【1】JSP第一次运行的时候会编译成Servlet，驻留在内存中以供调用。
【2】JSP是web开发技术，Servlet是服务器端运用的小程序，我们访问一个JSP页面时，服务器会将这个JSP页面转变成Servlet小程序运行得到结果后，反馈给用户端的浏览器。
【3】Servlet相当于一个控制层再去调用相应的JavaBean处理数据,最后把结果返回给JSP。
【4】Servlet主要用于转向，将请求转向到相应的JSP页面。
【5】JSP更多的是进行页面显示，Servlet更多的是处理业务，即JSP是页面，Servlet是实现JSP的方法。
【6】Servlet可以实现JSP的所有功能，但由于美工使用Servlet做界面非常困难，后来开发了JSP。
【7】JSP技术开发网站的两种模式：JSP + JavaBean；JSP + Servlet + JavaBean（一般在多层应用中, JSP主要用作表现层,而Servlet则用作控制层,因为在JSP中放太多的代码不利于维护，而把这留给Servlet来实现,而大量的重复代码写在JavaBean中）。
【8】二者之间的差别就是，开发界面是JSP直接可以编写。
比如在JSP中写Table标记： `<table>[数据]</table>`；
Servlet需要加入：`out.println(“<table>[数据]</table>”)`。
JSP文件在被应用服务器(例如：Tomcat、Resin、Weblogic和Websphere),调用过之后，就被编译成为了Servlet文件。也就是说在网页上显示的其实是Servlet文件。Tomcat下面JSP文件编译之后生成的Servlet文件被放在了work文件夹下，JSP中的HTML代码在Servlet都被out出来，而JSP代码按照标签的不同会放在不同的位置。
【9】JSP中嵌入JAVA代码，而Servlet中嵌入HTML代码。
【10】在一个标准的MVC架构中，Servlet作为Controller接受用户请求并转发给相应的Action处理，JSP作为View主要用来产生动态页面，EJB作为Model实现你的业务代码。


# Session 与 Cookie
## Session的概念

Session 是存放在服务器端的，类似于Session结构来存放用户数据，当浏览器 第一次发送请求时，服务器自动生成了一个Session和一个Session ID用来唯一标识这个Session，并将其通过响应发送到浏览器。
当浏览器第二次发送请求，会将前一次服务器响应中的Session ID放在请求中一并发送到服务器上，服务器从请求中提取出Session ID，并和保存的所有Session ID进行对比，找到这个用户对应的Session。

一般情况下，服务器会在一定时间内（默认30分钟）保存这个 Session，过了时间限制，就会销毁这个Session。
在销毁之前，程序员可以将用户的一些数据以Key和Value的形式暂时存放在这个 Session中。
当然，也有使用数据库将这个Session序列化后保存起来的，这样的好处是没了时间的限制，坏处是随着时间的增加，这个数据库会急速膨胀，特别是访问量增加的时候。
一般还是采取前一种方式，以减轻服务器压力。

## Session的客户端实现形式（即Session ID的保存方法）

一般浏览器提供了两种方式来保存，还有一种是程序员使用html隐藏域的方式自定义实现：

[1] 使用Cookie来保存，这是最常见的方法，本文“记住我的登录状态”功能的实现正式基于这种方式的。服务器通过设置Cookie的方式将Session ID发送到浏览器。如果我们不设置这个过期时间，那么这个Cookie将不存放在硬盘上，当浏览器关闭的时候，Cookie就消失了，这个Session ID就丢失了。如果我们设置这个时间为若干天之后，那么这个Cookie会保存在客户端硬盘中，即使浏览器关闭，这个值仍然存在，下次访问相应网站时，同 样会发送到服务器上。

[2] 使用URL附加信息的方式，也就是像我们经常看到JSP网站会有`aaa.jsp?JSESSIONID=*`一样的。这种方式和第一种方式里面不设置Cookie过期时间是一样的。

[3] 第三种方式是在页面表单里面增加隐藏域，这种方式实际上和第二种方式一样，只不过前者通过GET方式发送数据，后者使用POST方式发送数据。但是明显后者比较麻烦。

## cookie与session的区别

**cookie数据保存在客户端，session数据保存在服务器端。**

简单的说，当你登录一个网站的时候，如果web服务器端使用的是session,那么所有的数据都保存在服务器上面，客户端每次请求服务器的时候会发送 当前会话的sessionid，服务器根据当前sessionid判断相应的用户数据标志，以确定用户是否登录，或具有某种权限。
由于数据是存储在服务器 上面，所以你不能伪造，但是如果你能够获取某个登录用户的sessionid，用特殊的浏览器伪造该用户的请求也是能够成功的。
sessionid是服务器和客户端链接时候随机分配的，一般来说是不会有重复，但如果有大量的并发请求，也不是没有重复的可能性,显示别人的信息。

如果浏览器使用的是 cookie，那么所有的数据都保存在浏览器端，比如你登录以后，服务器设置了 cookie用户名(username),那么，当你再次请求服务器的时候，浏览器会将username一块发送给服务器，这些变量有一定的特殊标记。
服务器会解释为 cookie变量。所以只要不关闭浏览器，那么 cookie变量便一直是有效的，所以能够保证长时间不掉线。
如果你能够截获某个用户的 cookie变量，然后伪造一个数据包发送过去，那么服务器还是认为你是合法的。
所以，使用 cookie被攻击的可能性比较大。如果设置了的有效时间，那么它会将 cookie保存在客户端的硬盘上，下次再访问该网站的时候，浏览器先检查有没有 cookie，如果有的话，就读取该 cookie，然后发送给服务器。如果你在机器上面保存了某个论坛 cookie，有效期是一年，如果有人入侵你的机器，将你的 cookie拷走，然后放在他的浏览器的目录下面，那么他登录该网站的时候就是用你的的身份登录的。所以 cookie是可以伪造的。
当然，伪造的时候需要主意，直接copy cookie文件到 cookie目录，浏览器是不认的，他有一个index.dat文件，存储了 cookie文件的建立时间，以及是否有修改，所以你必须先要有该网站的 cookie文件，并且要从保证时间上骗过浏览器。

Session是由应用服务器维持的一个服务器端的存储空间，用户在连接服务器时，会由服务器生成一个唯一的SessionID,用该SessionID 为标识符来存取服务器端的Session存储空间。
而SessionID这一数据则是保存到客户端，用Cookie保存的，用户提交页面时，会将这一 SessionID提交到服务器端，来存取Session数据。这一过程，是不用开发人员干预的。所以一旦客户端禁用Cookie，那么Session也会失效。

服务器也可以通过URL重写的方式来传递SessionID的值，因此不是完全依赖Cookie。如果客户端Cookie禁用，则服务器可以自动通过重写URL的方式来保存Session的值，并且这个过程对程序员透明。

可以试一下，即使不写Cookie，在使用request.getCookies();取出的Cookie数组的长度也是1，而这个Cookie的名字就是JSESSIONID，还有一个很长的二进制的字符串，是SessionID的值。

## Session与Cookie区别和联系：

Cookies是属于Session对象的一种。但有不同，Cookies不会占服务器资源，是存在客服端内存或者一个cookie的文本文件中；而“Session”则会占用服务器资源。所以，尽量不要使用Session，而使用Cookies。但是我们一般认为cookie是不可靠的，session是可靠地，但是目前很多著名的站点也都以来cookie。有时候为了解决禁用cookie后的页面处理，通常采用url重写技术，调用session中大量有用的方法从session中获取数据后置入页面。

## Cookies与Session的应用场景：
Cookies的安全性能一直是倍受争议的。虽然Cookies是保存在本机上的，但是其信息的完全可见性且易于本地编辑性，往往可以引起很多的安全问题。所以Cookies到底该不该用，到底该怎样用，就有了一个需要给定的底线。


登陆验证信息。一般采用Session(“Logon”)＝true or false的形式。
用户的各种私人信息，比如姓名等，某种情况下，需要保存在Session里
需要在页面间传递的内容信息，比如调查工作需要分好几步。每一步的信息都保存在Session里，最后在统一更新到数据库。

cookie最典型的应用是：

（一）：判断用户是否登陆过网站，以便下次登录时能够直接登录。如果我们删除cookie，则每次登录必须从新填写登录的相关信息。

（二）：另一个重要的应用是“购物车”中类的处理和设计。用户可能在一段时间内在同一家网站的不同页面选择不同的商品，可以将这些信息都写入cookie，在最后付款时从cookie中提取这些信息，当然这里面有了安全和性能问题需要我们考虑了。
# 分布式session
##  session共享
原理是将session保存到分布式缓存数据库中如：redis， memcache等，然后多个服务器tomcat每次请求都通过NoSql数据库查询，如果存在，则获取值；反之存放值。
通过redis来实现session的共享，其主要有一下两种方法：
1、通过tomcat服务器的拓展功能实现
    这种方式比较简单，主要是通过继承session的ManagerBase类，实现重写session相关的方法，这种比较简单，
2、通过filter拦截request请求实现
    下面主要介绍这样实现方式：
    （1）写HttpSessionWrapper实现HttpSession接口，实现里面session相关的方法。
    （2）写HttpServletRequestWrapper继承javax.servlet.http.HttpServletRequestWrapper类，重写对于session  相关的方法。
    （3）写SessionFilter拦截配置的请求url，过去cookie中
    的sessionId，如果为空，对此次请求重写生成一个新的sessionId，在sessionId构造新的HttpServletRequestWrapper对象。
    （4）写SessionService实现session到redis的保存和过去，其key为sessionId，value为session对于的Map。
http://blog.csdn.net/fengshizty/article/details/50578639

## 粘性session

原理：粘性Session是指将用户锁定到某一个服务器上，比如上面说的例子，用户第一次请求时，负载均衡器将用户的请求转发到了A服务器上，如果负载均衡器设置了粘性Session的话，那么用户以后的每次请求都会转发到A服务器上，相当于把用户和A服务器粘到了一块，这就是粘性Session机制。

优点：简单，不需要对session做任何处理。

缺点：缺乏容错性，如果当前访问的服务器发生故障，用户被转移到第二个服务器上时，他的session信息都将失效。

适用场景：发生故障对客户产生的影响较小；服务器发生故障是低概率事件。
## 服务器session复制

原理：任何一个服务器上的session发生改变（增删改），该节点会把这个 session的所有内容序列化，然后广播给所有其它节点，不管其他服务器需不需要session，以此来保证Session同步。

优点：可容错，各个服务器间session能够实时响应。

缺点：会对网络负荷造成一定压力，如果session量大的话可能会造成网络堵塞，拖慢服务器性能。

# Cookie相关
## 跨域访问cookie
具体思路：在a.com下设置cookie后，嵌入一个iframe框链接b.com的页面，b.com设置好页面cookie后，再嵌入一个a.com的页面，然后通过parent.parent就可以调用最外层的a.com的js方法，从而进行跳转或者一些其它的操作。
http://www.tuicool.com/articles/E3iUva

或者可以再建一个系统，支持多个域名访问的，这个从一个域名下面获取到另一个域名的session、cookie，再写回去。
比如tomcat、apache等。

## Cookie防止xxs
### XSS攻击
XSS(Cross Site Scripting)是跨站点脚本攻击的缩写. 其就是利用站点开放的文本编辑并发布的功能, 从而造成攻击.
其实说的简单一点, 就是输入javascript脚本, 窃取并投递cookie信息到自己的站点.

### Cookie劫持的防
基于XSS攻击, 窃取Cookie信息, 并冒充他人身份.
给Cookie添加`HttpOnly`属性, 这种属性设置后, 只能在http请求中传递, **在脚本中, document.cookie无法获取到该Cookie值**. 对XSS的攻击, 有一定的防御值. 但是对网络拦截, 还是泄露了.
```
Set-Cookie: =[; =]
[; expires=][; domain=]
[; path=][; secure][; HttpOnly]
```
