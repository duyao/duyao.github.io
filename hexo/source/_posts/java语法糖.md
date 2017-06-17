title: java语法糖
toc: true
date: 2017-02-24 14:08:13
tags: jvm
categories: note
---
语法糖（Syntactic Sugar），也称糖衣语法，是由英国计算机学家Peter.J.Landin发明的一个术语，指在计算机语言中添加的某种语法，这种语法对语言的功能并没有影响，但是更方便程序员使用。
Java中最常用的语法糖主要有泛型、变长参数、条件编译、自动拆装箱、内部类等。
**虚拟机并不支持这些语法，它们在编译阶段就被还原回了简单的基础语法结构**，这个过程成为解语法糖。

## 泛型与泛型擦除

泛型实际上只在程序源码中存在，**在编译后的字节码文件中，就已经被替换为了原来的原生类型，并且在相应的地方插入了强制转型代码**。
因此对于运行期的Java语言来说，ArrayList<String>和ArrayList<Integer>就是同一个类。
所以泛型技术实际上是Java语言的一颗语法糖，Java语言中的泛型实现方法称为类型擦除，基于这种方法实现的泛型被称为伪泛型。

```java
public class FanxingTest{  
    public void method(List<String> list){  
        System.out.println("List String");  
    }  
    public void method(List<Integer> list){  
        System.out.println("List Int");  
    }  
}  
```

当我用Javac编译器编译这段代码时，报出了如下错误：
```
FanxingTest.java:3: 名称冲突：method(java.util.List<java.lang.String>) 和 method

(java.util.List<java.lang.Integer>) 具有相同疑符

       public void method(List<String> list){

                   ^

FanxingTest.java:6: 名称冲突：method(java.util.List<java.lang.Integer>) 和 metho

d(java.util.List<java.lang.String>) 具有相同疑符

       public void method(List<Integer> list){

                   ^

2 错误
```

这是因为泛型List<String>和List<Integer>编译后都被擦除了，变成了一样的原生类型List，擦除动作导致这两个方法的特征签名变得一模一样，在Class类文件结构一文中讲过，Class文件中不能存在特征签名相同的方法。

```java
public class FanxingTest{  
    public int method(List<String> list){  
        System.out.println("List String");  
        return 1;  
    }  
    public boolean method(List<Integer> list){  
        System.out.println("List Int");  
        return true;  
    }  
}
```
修改后发现这时编译可以通过了（注意：Java语言中true和1没有关联，二者属于不同的类型，不能相互转换，不存在C语言中整数值非零即真的情况）。两个不同类型的返回值的加入，使得方法的重载成功了。

Java代码中的方法特征签名只包括了方法名称、参数顺序和参数类型，并不包括方法的返回值，因此方法的返回值并不参与重载方法的选择，这样看来为重载方法加入返回值貌似是多余的。对于重载方法的选择来说，这确实是多余的，但我们现在要解决的问题是让上述代码能通过编译，让两个重载方法能够合理地共存于同一个Class文件之中，这就要看字节码的方法特征签名，它不仅包括了Java代码中方法特征签名中所包含的那些信息，还包括方法返回值及受查异常表。为两个重载方法加入不同的返回值后，因为有了不同的字节码特征签名，它们便可以共存于一个Class文件之中。

注：
（A）、Java语言层面的方法特征签名：
     特征签名 = 方法名 + 参数类型 + 参数顺序；
（B）、JVM层面的方法特征签名：
     特征签名 = 方法名 + 参数类型 + 参数顺序 + 返回值类型；

## 自动装箱拆箱
```java
//装箱就是  自动将基本数据类型转换为包装器类型；拆箱就是  自动将包装器类型转换为基本数据类型。
//主要是看等号左边的定义的类型
public static void main(String[] args) {
//		Integer i1 = 59;
//		int i2 = 59;
//		Integer i3 = Integer.valueOf(59);
//		Integer i4 = new Integer(59);
	Integer i1 = 111111;
	int i2 = 111111;
	Integer i3 = Integer.valueOf(111111);
	Integer i4 = new Integer(111111);
	//当为59时，1-3都是true
	//因为jvm对于小于1字节的以下的整数都会加载进内存，除非用new Integer()显示声明新建对象
	//因此1-3都是指向同一个对象,因此12真，4假
	//而3是对于int进行比较值的大小
	//当为111111时，13true，24false
	//i3的valueof会先判断数字是都小于1字节(127)，若小于不产生新对象，大于127则产生新对象
	System.out.println(i1 == i2);
	System.out.println(i1 == i3);
	//24和23何时都相等，因为2是int3是Integer，因为一个是基本类型，一个是封装类
	System.out.println(i2 == i4);
	System.out.println(i3 == i4);

	System.out.println(i2 == i3);
	//无论何时14不等1与4是不同的对象，因为4被new，虽然他们都是integer
	System.out.println(i1 == i4);


  //Integer、Short、Byte、Character、Long这几个类的valueOf方法的实现是类似的。
  /*
  public static Integer valueOf(int i) {
      if (i >= IntegerCache.low && i <= IntegerCache.high)
          return IntegerCache.cache[i + (-IntegerCache.low)];
      return new Integer(i);
  }
   */

  //Double、Float的valueOf方法的实现是类似的。
  /*
  public static Double valueOf(double d) {
      return new Double(d);
  }
  */
	Double d1 = 34.0;
	Double d2 = 34.0;
	System.out.println("d1 == d2  ->  " + (d1 == d2)); //false


	//当 "=="运算符的两个操作数都是 包装器类型的引用，则是比较指向的是否是同一个对象，
	//而如果其中有一个操作数是表达式（即包含算术运算）则比较的是数值（即会触发自动拆箱的过程）
	Long ll = 3l;
	int a = 1;
	int b = 2;
//		long b = 2;
	System.out.println(ll == (a + b)); //true
	//对于包装器类型，equals方法并不会进行类型转换,比较是否类型相同
	System.out.println(ll.equals(a + b)); //b是int结果是false，b是long结果是true


  //byte运算
  //加数是常量，先运算后赋值
  //加数是变量，先转类型，再运算
  byte b1=1,b2=2,b3,b6;
	final byte b4=4,b5=6;
	//b4b5是byte相加计算后升为int，而其实final，因此编译时就已经是10了
	b6=b4+b5;
	//b1+b2计算时候提升为int，而b3是byte不是同一类型，会报错，需要强制转换
//		b3=(b1+b2);
//		System.out.println(b3+b6);
	System.out.println(b6);

	Byte b7  = 8+2;
	System.out.println(b7);


}
```
