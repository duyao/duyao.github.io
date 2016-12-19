title: Scala
toc: true
date: 2016-10-13 09:57:50
tags:
categories:
---

## 重要概念
- 纯函数(pure function)
无副作用的函数
- 函数的纯粹性(purity)
- 无副作用(side effect)
无状态mutation的变化
```
int x;
public int f1(int y){
	return x+y;
}
public int f2(int y){
	x = x+y;
	return x+y;
}
```
这里f1是无副作用，因为没有改变x的值，而f2具有副作用，改变了x的值
- 引用透明性(referential transparency)
相同的输入，总是得到相同的输出
```
StringBuffer x = "hello";
StringBuffer y = x.append("world");
StringBuffer z = x.append("world");
```
这里`append`函数不具有引用透明性，因为y和z的输出并不相同
- 不变形(immutability)
为了获得引用透明性，任何值都不会引起不变化
- 函数是一等公民first-class function
- 高阶函数high order function
- 闭包closure
- 表达式求值策略-严格求值和非严格求值
一切都是表达式
call by value VS call by name
- 惰性求值lazy evaluation

## 数据类型

## 语法
- val 常量
- var 变量
- lazy val 懒加载常量
- 字符串插值
使用`s+${}`
```
val s : String = "hello";
s"${s} world";
```
- 函数
```
def functionName (param : ParamType)[: ReturnType]   = {
	//function body
}
```
```
例子
def hello1(name:  String) = {
	s"hell0 ${name}"
}
```
- if表达式
`if(logical_exp) valA else valB`
```
val a = 1;       a: Int = 1
if (a == 1) a     res4: AnyVal = 1  
if (a != 1) "not one"     res5: Any = ()  
if (a != 1) "not one" else 1       res6: Any = 1
```

- for comprehension
```
val l = List("a", "b", "c")

for {
	s <- l  //generator
	if (s.length < 3) //filter
} println(s)

val result_for = for {
	s <- l
	sup = s.toUpperCase();  //variable binding
	if (sup != "")
} yield (sup) //generate new collection

```


- try, catch, finally
```
try {
	Integer.parseInt("dog")
} catch {
	case _ => 0; //_是通配符，=>赋值
} finally {
	println("always be printed")
}
```

- match
```
exp match{
	case p1 => val1
	case p2 => val2
	...
	case _ => valn
}
```
```
val code = 4
code match {
	case 1 => "one"
	case 2 => "two"
	case _ => "others"
}
```


## 求值策略
`call by value` 一般都是=使用，调用就计算
`call by name`  调用不计算，只有等到执行才会计算

```
def bar(x : Int, y : => Int) = 1
def loop():Int = loop
bar(1,loop)   //res2: Int = 1
bar(loop,1)   //死循环
```
## 高阶函数

用函数作为形参或返回值的函数成为高阶函数

### 匿名函数
匿名函数anonymous function就是函数常量，也成为函数文字量function literal
在scala中匿名函数的定义格式为`(形参列表) => {函数体}`
```
def operate(f:(Int,Int) => Int) ={
	f(4,4)
}    //> operate: (f: (Int, Int) => Int)Int
def tr(x:Int,y:Int)=x+y  //tr: (x: Int, y: Int)Int
operate(tr)   //res0: Int = 8

def greeting() =(name:String) => {"hello"+" "+name}
greeting() ("123")
```

### 柯里化 curried functoin
把具有多个参数的函数转化为一条函数链，每个节点上是单一参数
```
def curriedAdd(a:Int)(b:Int)=a+b  //curriedAdd: (a: Int)(b: Int)Int
curriedAdd(2)(2)  //res1: Int = 4
val addOne = curriedAdd(1)_  //addOne: Int => Int = <function1>
addOne(2)  //res2: Int = 3
```
上面的例子可以看出来，柯里化可以复用函数


### 递归函数recuritive function
```
def f(n: Int): Int =
	if (n < 0) 1
	else n * f(n - 1)

f(5)

@annotation.tailrec
def f(n: Int, m: Int): Int =
	if (n <= 0) m
	else f(n - 1, m*n)

f(5,1)
```
上面的m是计算的最新结果

### 例子

$$\sum_{i=a}^b f(x)=0$$
求a到b的区间和

```
def sum(f: Int => Int)(a: Int)(b: Int): Int = {
  def loop(n: Int, acc: Int): Int = {
    if (n > b) {
      println(s"n=${n}, acc=${acc}")
      acc
    } else {
      println(s"n=${n}, acc=${acc}")
      loop(n + 1, acc + f(n))
    }
  }
  loop(a,0)
}
sum(x=>x)(1)(5)
sum(x=>x*x)(1)(5)
```
打印结果
```
n=1, acc=0
n=2, acc=1
n=3, acc=3
n=4, acc=6
n=5, acc=10
n=6, acc=15
res0: Int = 15
n=1, acc=0
n=2, acc=1
n=3, acc=5
n=4, acc=14
n=5, acc=30
n=6, acc=55
res1: Int = 55
```


