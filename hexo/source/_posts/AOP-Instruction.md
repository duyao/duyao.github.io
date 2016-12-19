title: AOP_Instruction
date: 2015-10-18 21:34:45
tags: spring
categories: note
---

## `cross-cutting concerns`
System services such as logging, transaction management, and security often find their way into components whose core responsibilities is something else. 
These system services are commonly referred to as `cross-cutting concerns` because they tend to cut across multiple components in a system.

## AOP
Using AOP, system-wide concerns blanket the components they impact. This leaves the application components to focus on their specific business functionality


That’s `boilerplate code—the code that you often
have to write over and over again to accomplish common and otherwise simple tasks.

## Terms of AOP
|Terms|	Description|
|:--:|:--|
|Aspect	|A module which has a set of APIs providing cross-cutting requirements. For example, a logging module would be called AOP aspect for logging. An application can have any number of aspects depending on the requirement.|
|Join point	|This represents a point in your application where you can plug-in AOP aspect. You can also say, it is the actual place in the application where an action will be taken using Spring AOP framework.
|Advice	|This is the actual action to be taken either before or after the method execution. This is actual piece of code that is invoked during program execution by Spring AOP framework.|
|Pointcut|	This is a set of one or more joinpoints where an advice should be executed. You can specify pointcuts using expressions or patterns as we will see in our AOP examples.|




## Types of Advice

Spring aspects can work with five kinds of advice mentioned below:

|Advice|	Description|
|:--:|:--|
|before	|Run advice before the a method execution.|
|after	|Run advice after the a method execution regardless of its outcome.|
|after-returning|	Run advice after the a method execution only if method completes successfully.|
|after-throwing|	Run advice after the a method execution only if method exits by throwing an exception.|
|around|	Run advice before and after the advised method is invoked.|







