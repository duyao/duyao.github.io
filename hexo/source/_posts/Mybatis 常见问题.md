title: Mybatis 常见问题
date: 2015-10-15 20:34:22
tags: Mybatis
categories: note
---

# Mybatis 常见问题


## resultMap与resultType
两者都是用来实现数据库与java代码的映射。

resultMap必须写指明映射关系，且要select等语句要用其id
![ResultMap][1]

reslutType无需指明关系，映射方式是看语句的列名与java代码列名相同，且忽略大小写,内容是java的文件名
```xml
<select id="queryMessageList" parameterType="com.imooc.bean.Message" resultType="com.imooc.bean.Message">
  select ID,COMMAND,DESCRIPTION,CONTENT from MESSAGE
</select>
```
<!--more-->
## parameterMap与parameterType
两者都是指定参数内容
parameterMap已经废弃
parameterType是指明参数类型，如果是string和java常用类型，直接写名字即可，比如`int`,`double`,`string`，如果是自定义和其他类型，需要写出报名，比如`com.imooc.bean.Message`,`java.util.List`

```xml
<delete id="deleteBatch" parameterType="java.util.List" >
  delete from message where ID in (
  	<foreach collection="list " item="item" separator=",">
  		 #{item}
  	</foreach>
  )
</delete>
```

## `#{}`与`${}`
两者都是传入参数的
`#{}`是带有预编译过程的，而`${}`没有，传进来什么就是什么
```
select ID,COMMAND,DESCRIPTION,CONTENT from MESSAGE where ID = #{id}
select ID,COMMAND,DESCRIPTION,CONTENT from MESSAGE where ID = ${id}
```
编译后变为
```
select ID,COMMAND,DESCRIPTION,CONTENT from MESSAGE where ID = '?'
select ID,COMMAND,DESCRIPTION,CONTENT from MESSAGE where ID = xxx
```
可以看出`#{}`可以有预编译，防止sql注入等，而`${}`原样带入
但是在`order by`语句中经常使用`${}`
```
select ID,COMMAND,DESCRIPTION,CONTENT from MESSAGE order by ${id}
```

## 获取自增主键的值
当主键自增，添加数据需要获取主键的值时可使用时`useGeneratedKeys="true"`表示逐渐自增和` keyColumn="id"`表示获取`id`的值
```
<insert id="insert" parameterType="UserAlias" useGeneratedKeys="true" keyColumn="id">
INSERT INTO user( id,username,) VALUES ( #{id},#{username})
</insert>
```
## 中文乱码
- 数据库文件
- Mybatis的configuration.xml文件
```xml
<property name="url" value="jdbc:mysql://127.0.0.1:3306/wechat?useUnicode=yes&characterEncoding=UTF-8"/>
```
- jsp文件
```jsp
<%@ page language="java" import="java.util.*" pageEncoding="utf-8"%>
```
- servlet文件

```java
// 设置编码
request.setCharacterEncoding("utf-8");
response.setContentType("text/html;charset=utf-8");
```
- Myeclipse文件编码
  






  [1]: http://7xilc8.com1.z0.glb.clouddn.com/Mybatis_ResultMap.png