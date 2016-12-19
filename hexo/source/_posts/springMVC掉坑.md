title: springMVC掉坑
date: 2015-10-29 20:12:43
tags: springMVC 
categories: note
---
本文记录使用springmvc时候的困惑，及解决的方法

<!--more-->

## 坑：`web.xml`文件的version

开始建立项目时候新建`mavenProject`，此时默认的`Dynamic Web Module`是2.3，
但是在2.4版本以下默认关闭el表达式，而且项目一般都没有`src/main/java`等source folder,`jsp`文件有错
因此`jsp`文件中`${}`内容全不出现

## 解决

建立`web project`可以选择`Dynamic Web Module`然后添加`maven`，上面的问题都不会出现的

---

## 坑：对应的servlet.xml

不是个都叫HelloWeb-servelt.xml!!!!
不是个都叫HelloWeb-servelt.xml!!!!
不是个都叫HelloWeb-servelt.xml!!!!
仔细看文档啊啊啊啊！！！

>The web.xml file will be kept WebContent/WEB-INF directory of your web application. OK, upon initialization of [servlet-name] DispatcherServlet, the framework will try to load the application context from a file named [servlet-name]-servlet.xml located in the application's WebContent/WEB-INF directory.

`servlet-name`就是包名

---

## 生成项目的url是谁？

建立时候时，是根据`maven`中的`Artifact id`确定项目的url
因此在配置`web.xml`时，注意`servlet`名字要与`Artifact id`相同，且配置的`[servlet]-servlet.xml`要与其对应
也就是上面的问题



