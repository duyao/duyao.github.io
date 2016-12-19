title: Spring IOC 容器
date: 2016-02-17 09:45:34
tags: spring
---

## 接口

- 用于沟通的中介物的抽象化

- 实体把自己提供给外界的一种抽象化说明，用来由内部操作分离出外部沟通方法，使其能被修改内部而不影响外界其他实体与其交互的方式

- java接口即声明，声明那些方法是对外公开的

## 面向接口编程

- 结构设计中，分清层次及其调用关系，每层只能向外(上层)提供一组功能接口，各层件仅依赖接口而非实现类

- 接口实现的变动不影响各层间的调用，这一点在公共服务中尤为重要

- `面向接口编程`中的`接口`式用于隐藏具体实现和实现多态的组件

## IOC

- IOC
**控制反转**，控制权的转移，应用程序本身不负责雨来对象的从创建和维护，而是有外部容器负责创建和维护

- DI
**依赖注入**，是其一种实现方式

- 目的
创建对象并且组装对象之间的关系

- 获得依赖对象的过程被反转，即获得依赖对象的过程由自身管理变为了由IOC容器主动注入，即IOC容器运行期间，动态的将某种依赖关系注入到对象之中

## Bean容器的初始化

### 基础：两个包

- org.springframework.beans
- org.springframework.context
- BeanFactory提供配置结构和基本功能，加载并初始化Bean
- ApplicationContext保存了Bean对象并在Spring中被广泛使用

### 方式,ApplicationContext

- 本地文件
- Classpath
- Web应用中依赖的servelt或Listener

## Spring注入

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




