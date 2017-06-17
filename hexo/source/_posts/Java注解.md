title: Java注解
toc: true
date: 2017-05-12 18:59:43
tags: java
categories: note
---
# 分类
## 源码注解
只存在于源码中，编译时就没有了，即class文件无注解
## 编译时注解
注解在源码和class文件都在,比如`Override`
## 运行时注解
运行阶段起作用，甚至影响运行时的逻辑，比如`Autowired`

# 自定义注解
## 语法要求
必须使用`@interface`关键字
成员函数无参数无异常，同时可以指定默认值，形式类似于接口声明方法
成员类型是受限制的，只能是原始数据类型或者String、Class、Annotation、Enumerattion
如果只有一个成员，那么成员必须是`String value();`
```java
//作用域
@Target({ElementType.METHOD,ElementType.FIELD})
//声明周期
@Retention(RetentionPolicy.RUNTIME)
//可以继承，但是只用类，不用于接口
@Inherited
//生成javadoc中存在该注解
@Documented
public @interface Description {
    String desc = null;
    String author();
    int age() default 18;
}
//使用
@Description(desc="desc",author="amy")
public void fun {
    String desc = null;
    String author();
    int age() default 18;
}
```
