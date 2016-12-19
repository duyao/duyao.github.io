title: jsp原理
date: 2015-10-16 09:48:39
tags: jsp
categories: note
---

jsp(java server page)
:   
    
    * 运行在服务器端
    * 基础是servlet，因此也是动态网页
    * 是一种综合技术=html+css+javasricp+java片段+jsp标签
    * jsp无需配置，修改后无需重新发布
    * 访问方法：http://ip:8080/web应用名/jsp路径(路径从webRoot开始)
### 原理
#### web服务器是如何调用并执行jsp页面的
例如文件为`myjsp.java`
第一次访问jsp文件，服务器会将文件翻译为servlet文件`myjsp_jsp.java`并编译为`.class`文件，在文件tomcat根目录的`..\work\Catalina`中可以找到，即`D:\Program Files\apche\apache-tomcat-6.0.43\work\Catalina\localhost\web应用名\org\apache\jsp`，然后加载到服务器内存中去，如果是第二次访问，就会直接访问内存中的实例，因此jsp也是单例的，因此第一访问jsp网站速度比较慢，以后访问速度比较快。如果某个jsp被修改了，就相当于第一次访问jsp。(web服务器有一张表记录每个jsp是否是第一次访问)
**注意：jsp出错后会报出翻译后servlet中的位置**
在该`xxx_jsp.java`文件中，有几分主要函数如下，其实就是servlet中的需要实现的函数`init`,`destory`,`service`

<!--more-->

```java
//myjsp_jsp.java
public final class jsp1_jsp {
    
  public void _jspInit() {
    ...
  }
  public void _jspDestroy() {
  }
  public void _jspService(HttpServletRequest request, HttpServletResponse response)
        throws java.io.IOException, ServletException {
    ...
    }
```

#### jsp的html标签的排版空间,java代码段是如何发送到客户端的
```java
//myjsp_jsp.java的_jspService函数
public void _jspService(HttpServletRequest request, HttpServletResponse response) throws java.io.IOException, ServletException {

    PageContext pageContext = null;
    HttpSession session = null;
    ServletContext application = null;
    ServletConfig config = null;
    JspWriter out = null;
    Object page = this;
    
    
    //这里out已经设置为内部流(JspWriter out = null)，因此可以在jsp中直接使用
    //html的标签直接使用out输出
    out.write("<html>\r\n");
    out.write(" <head>\r\n");
    out.write(" </head>\r\n");
    out.write(" <body>\r\n");
    out.write("   <h1>hello,world!</h1>\r\n");
    out.write("</html>");
    //对于内嵌的java代码则定义为内部变量,放在_jspService函数中
    /*jsp中的源代码
    <%
    int i=1;
    out.print("i="i);
    %>
    */
    int i=1;
    out.print("i="i);

```

#### web服务器调用jsp时会哪些java对象
一共提供了9个，在生成的`xxx_jsp.java`文件中`_jspService函数`中可以找到

* out
向客户端输出字节流媒体
> out.print();
* request
接受客户端的http请求，相当于servlet中的`HttpServletRequest`
```java
request.getParameter(String);
request.getParameterValues(String);
request.getAttribute(String);
request.getAttributeNames();
request.getCookies();
```

* response
封装jsp产生的回应，相当于servlet的`HttpServletResponse `
```java
response.addCookie(Cookie);
response.sendRedirect(String);
```

* session
用于保存用户信息，跟踪用户行为，相当于servlet的`HttpSession`
```java
session.getAttribute(String);
session.setAttribute(String,Object);
```

* application
多个用户共享，保存信息，相当于servlet的`ServletContext`
```java
application.getAttribute(String);
application.setAttribute(String,Object);
```

* pageContext
只在本页面生效的域对象，可以setAttribute等

* exception
代表运行时的一个异常

* page
代表jsp这个实例，相当于`this`

* config
得到`web.xml`的配置信息，相当于servlet的`ServletConfig`

#### jsp的4个作用域
| 名称	　|　作用域	|
| -------| -------|
| application|在所有应用程序中有效|	
| session	　|　在当前会话中有效	|
| request |	　　在当前请求中有效	|
| page	　|　在当前页面有效 |	


---

### jsp语法

#### 指令元素
概念：用于从jsp发送一个信息到容器，比如设置全局变量，文字编码，引入宝

* page指令
`<%@ page language="java" import="java.util.*" pageEncoding="ISO-8859-1"%>`

    * language="java" 放入的语言
    * import="java.util.*" 引入了哪些包
    * pageEncoding="utf-8" 以什么方式翻译为servlet
    * contentType="text/html; charset=utf-8" 以什么方式显示网页
    * isErrorPage="/err"

* include指令
`<%@include file="index.jsp" %>`
该指令用于引入一个文件，jsp引擎通常会把这2个页面翻译成一个jsp页面，也成为静态引入，以`webRoot`为根目录
**被引入的jsp页面只保留page指令，标签`<html>`,`<body>`均要省略**

* taglib指令

#### 脚本元素

* scrplet 
`<% java代码 %>`
* 表达式
相当于`_jspService`的变量
`<%=java表达式%>`
例如`<%=rs.getInt(1)%>`
* declaration声明
相当于`xxx——jsp.java`的成员变量和成员函数
`<%! java代码 %>`,`<%! java函数代码 %>`
与表达式声明的区别是，表示声明是局部变量，declaration是全局变量
而函数定义只能用在declaration中定义，不能再表达式式中定义
```jsp
<%! int i=900; //全局变量 %>
<% int i=90; //局部变量
  out.print("i="+i); 
%>
```
在对相应的`xx_jsp.java`文件中声明的位置完全不同，declaration变量时该servlet的成员变量
```java
//jsp1_jsp.java
public final class jsp1_jsp {
    int i=900;//全局变量
    public void _jspInit() {
    ...
    }
    public void _jspDestroy() {
    }
    public void _jspService(HttpServletRequest request, HttpServletResponse response)throws java.io.IOException, ServletException {
    int i=90;//局部变量
    out.print("i="+i); 
    ...
    }
```

#### 动作元素
* `<jsp:forward >`
`<jsp:forward page="index.jsp"></jsp:forward>`
这里起到跳转作用，通常开发时，将各种jsp文件不直接放在`WebRoot`中，因为这样利用`<a>`标签可以将网站源代码下载下来，而且得到路径后可以直接访问，很不安全，因此要把jsp文件放在`/WebRoot/WEB-INF`文件夹中，这样即使得到路径也不能直接访问和下载源码。而访问网站的方法是`WebRoot`放一个`xxx.jsp`通过`<jsp:forward page="/WEB-INF/xxx.jsp"></jsp:forward>`转向`/WebRoot/WEB-INF`文件夹要访问的jsp文件

* `<jsp:include >`
`<jsp:include page=""></jsp:include>`
这里也是引入文件，注意与``<%@include>`区分，但是这里的引入是动态引入，即jsp引擎会将这2个servlet分别编译形成2个java文件，因此不必省略标签`<html>`,`<body>`

#### jsp注释
2种方式
`<-- 注释 -->`和`<%-- 注释 --%>`
没有%的可以在源码中显示，而有%的在源码中不能显示
建议使用`<%-- 注释 --%>`