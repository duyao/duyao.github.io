title: Mybatis 接口式编程
date: 2015-10-15 23:32:10
tags: Mybatis
categories: note
---

##引言
首先分析MessageDao中获取`sqlSession`的过程
```java
//Command是namespace
//queryCommandList是一种select方法的id
//command是传入的Command类型的参数
//list是该方法返回值
list = sqlSession.selectList("Command.queryCommandList",command);
```
由于以上信息都在`Command.xml`文件中，且都是手动填入，可能会出错。
因此引入接口式编程，即把该方法写成接口形式，这样调用时会校验，减少出错。
<!--more-->

## 实现-动态代理jdkProxy




