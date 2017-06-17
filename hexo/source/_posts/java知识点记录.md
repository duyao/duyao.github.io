title: java知识点记录
toc: true
date: 2017-02-20 19:10:05
tags: java
categories: note
---
# 类方法就是静态方法，方法内不能使用this关键字

# 比较问题
对于==号，如果两边是int型，则比较的数值
如果两边是Integer，则比较的是地址
如果一个是Integer，另一个是int，比较的是值，因为Integer会自动拆箱
```
Integer i1 = 59; //自动装箱，调用Integer.valueOf()，指向常量池中的59
int i2 = 59;
Integer i3 = Integer.valueOf(59); //Integer.valueOf()中，如果数字属于[-128,127]，那么不会产生新的对象，而是指向常量池中的59
Integer i4 = new Integer(59); //有new关键字，一定会产生一个新的对象


System.out.println(i1 == i3);//true，都会指向常量池中的59

//12,24和23何时都相等，因为2是int,1和3、4是Integer，因为一个是基本类型，一个是封装类,比较的是数值
System.out.println(i1 == i2);//true
System.out.println(i2 == i4);
System.out.println(i2 == i3);

//无论何时14不等1与4是不同的对象，因为4被new，虽然他们都是integer
System.out.println(i1 == i4);//false

//基本型封装类型调用equals(),但是参数是基本类型，这时候，先会进行自动装箱，基本型转换为其封装类型
int a = 1;
int b =2;
Integer c =3;
System.out.println(c.equals(a+b));//true
```
valueOf方法
```
public static Integer valueOf(int i) {
   if (i >= IntegerCache.low && i <= IntegerCache.high)
       return IntegerCache.cache[i + (-IntegerCache.low)];
   return new Integer(i);
}
```

加入byte
```
byte b1=1,b2=2,b3,b6,b8;
final byte b4=4,b5=6,b7;
b3=(b1+b2);  /*语句1*/
b6=b4+b5;    /*语句2*/
b8=(b1+b4);  /*语句3*/
b7=(b2+b5);  /*语句4*/
System.out.println(b3+b6);
```
这里只有语句1正确，因为右边都是final，所以编译时就是int了
其余的语句因为是有变量的，所以int+byte=byte的，需要类型转换

# list与array互相转化

```
public String[] toArray(List<String> list){
    String [] strings = new String[list.size()];
    return  list.toArray(strings);
}
public List<String> toArray(String[] strings){
    return  Arrays.asList(strings);
}
```
但是当list转化为int数组就不可以，因为list中是包装类，而返回时基本类型
```
public int[] toArray(List<Integer> list){
    int [] ints = new int[list.size()];
    //ERROR: toArray (java.lang.Object[]) in List cannot be applied to (int[])
    return  list.toArray(ints);

    Integer [] ints = new Integer[list.size()];
    //ERROR: toArray (T[]) in List cannot be applied to (Integer[])
    return  list.toArray(ints);
}
```

# 取反~
`-n = ~n + 1` 负数的补码等于原码取反加一

- enum枚举
```java
enum AccountType
{
    SAVING, FIXED, CURRENT;
    private AccountType()
    {
        System.out.println(“It is a account type”);
    }
}
class EnumOne
{
    public static void main(String[]args)
    {
        System.out.println(AccountType.FIXED);
    }
}
```
结果`It is a account type It is a account type It is a account type FIXED`
每个枚举都是实例，在实例化的时候都要调用AAA的构造器,因此要打印3次

# 反射中，Class.forName和classloader的区别
Java中class.forName()和classLoader都可用来对类进行加载。
**class.forName()除了将类的.class文件加载到jvm中之外，还会对类进行解释，执行类中的static块。**
而**classLoader只干一件事情，就是将.class文件加载到jvm中，不会执行static中的内容,只有在newInstance才会去执行static块。**
Class.forName(name, initialize, loader)带参函数也可控制是否加载static块。并且只有调用了newInstance()方法采用调用构造函数，创建类的对象
```java
//Class.forName(String className)  这是1.8的源码  
   public static Class<?> forName(String className) throws ClassNotFoundException {  
       Class<?> caller = Reflection.getCallerClass();  
       return forName0(className, true, ClassLoader.getClassLoader(caller), caller);  
   }  
//注意第二个参数，是指Class被loading后是不是必须被初始化。 不初始化就是不执行static的代码即静态代码  
```


LoadClass（）方法加载类及初始化过程：
类加载（loadclass（））（加载）——》newInstance（）（链接+初始化）
newInstance（）:
（开始连接）静态代码块——》普通变量分配准备（a=0;b=0;c=null）——》（开始初始化）普通变量赋值（a=1;b=2;c=”haha”）——》构造方法——》初始化成功。


Class.forName(Stirng className)一个参数方法加载类及初始化过程：
类加载(Class.forName())（加载）——》静态代码块——》newInstance（）（链接+初始化）

newInstance（）：
（开始连接）普通变量分配准备（a=0;b=0;c=null）——》（开始初始化）普通变量赋值（a=1;b=2;c=”haha”）——》构造方法——》初始化成功。


# 继承问题
在调用子类构造器之前，会先调用父类构造器

当子类构造器中没有使用"super(参数或无参数)"指定调用父类构造器时，是默认调用父类的无参构造器，如果父类中包含有参构造器，却没有无参构造器，则在子类构造器中一定要使用“super(参数)”指定调用父类的有参构造器，不然就会报错。
https://www.nowcoder.com/questionTerminal/af8cf04602e045958d13d16d20a1bf02
```java
class Test {
    public static void main(String [] args){
        System.out.println(new B().getValue());
    }
    static class A{
        protected int value;
        public A(int v) {
            setValue(v);
        }
        public void setValue(int value){
            this.value = value;
        }
        public int getValue(){
            try{
                value++;
                return value;
            } catch(Exception e){
                System.out.println(e.toString());
            } finally {
                this.setValue(value);
                System.out.println(value);
            }
            return value;
        }
    }
    static class B extends A{
        public B() {
            super(5);
            setValue(getValue() - 3);
        }
        public void setValue(int value){
            super.setValue(2 * value);
        }
    }
}
```
这里执行setValue是调用b的而不是a的，因为这里是new b
# Statement、PreparedStatement和CallableStatement

|接口|	推荐使用|
| :------: |  :------: |
|Statement	|使用通用访问数据库。当在运行时使用静态SQL语句。 Statement接口不能接受的参数。|
|PreparedStatement	|当计划多次使用SQL语句。 那么可以PreparedStatement接口接收在运行时输入参数。|
|CallableStatement	|当要访问数据库中的存储过程中使用。 CallableStatement对象的接口还可以接受运行时输入参数。|
（1）Statement：
每次执行sql语句，数据库都要执行sql语句的编译 ，最好用于仅执行一次查询并返回结果的情形，效率高于PreparedStatement.但存在sql注入风险

（2）PreparedStatement：
是预编译执行的。在执行可变参数的一条SQL时，PreparedStatement比Statement的效率高，因为DBMS预编译一条SQL当然会比多次编译一条SQL的效率要高。
安全性更好，有效防止sql注入问题。对于多次重复执行的语句，使用PreparedStament效率会更高一点，并且在这种情况下也比较适合使用batch。

执行的SQL语句中是可以带参数的,并支持批量执行SQL。由于采用Cache机制，则预先编译的语句，就会放在Cache中，下次执行相同SQL语句时，则可以直接从Cache中取出来。
例：
```
PreparedStatement pstmt  =  con.prepareStatement("UPDATE EMPLOYEES  SET name= ? WHERE ID = ?");
pstmt.setString(1, "李四");
pstmt.setInt(2, 1);
pstmt. executeUpdate();
```
但是对于in来说不是很友好，需要自己定义写多少个问号
```
String[] names = new String[]{'name1','name2','name3'};  
String sql = "select e.* from employee e where e.name in (?,?,?)";  
rs = pstmt.excuteQuery(sql);  
pstmt.setString(1,names[0]);  
pstmt.setString(2,names[1]);  
pstmt.setString(3,names[2]);  
```
（3）CallableStatement：
扩展 PreparedStatement接口，用来调用存储过程,它提供了对输出和输入/输出参数的支持。CallableStatement 接口还有对 PreparedStatement 接口提供的输入参数的sql查询的支持。


http://www.importnew.com/5006.html

# == equals
## ==
== : 它的作用是**判断两个对象的地址是不是相等**。即，判断两个对象是不试同一个对象。
对于基本类型使用==比如int和Integer比较，会要拆装箱，然后进行值比较大小
## equals
equals() 的作用是 用来判断两个对象是否相等。
equals() 定义在JDK的Object.java中。通过判断两个对象的地址是否相等(即，是否是同一个对象)来区分它们是否相等
```java
public boolean equals(Object obj) {
    return (this == obj);
}
```
但是java中很多类对于equals方法进行了重写。比如String中首先比较是否是同一个类型，比较字符串内容是否相等。
总结：
equals() : 它的作用也是判断两个对象是否相等。但它一般有两种使用情况：
情况1，类没有覆盖equals()方法。则通过equals()比较该类的两个对象时，等价于通过“==”比较这两个对象。
情况2，类覆盖了equals()方法。一般，我们都覆盖equals()方法来两个对象的内容相等；若它们的内容相等，则返回true(即，认为这两个对象相等)。
## hashCode()
hashCode() 的作用是获取哈希码，也称为散列码；它实际上是返回一个int整数。这个哈希码的作用是确定该对象在哈希表中的索引位置。

hashCode() 定义在JDK的Object.java中，这就意味着Java中的任何类都包含有hashCode() 函数，同时object中hashcode使用的是native方法。
虽然，每个Java类都包含hashCode() 函数。但是，仅仅当创建并某个“类的散列表”(关于“散列表”见下面说明)时，该类的hashCode() 才有用(作用是：确定该类的每一个对象在散列表中的位置；其它情况下(例如，创建类的单个对象，或者创建类的对象数组等等)，类的hashCode() 没有作用。
上面的散列表，指的是：Java集合中本质是散列表的类，如HashMap，Hashtable，HashSet。
也就是说：hashCode() 在散列表中才有用，在其它情况下没用。在散列表中hashCode() 的作用是获取对象的散列码，进而确定该对象在散列表中的位置。

hashCode() 和 equals() 的关系
第一种 不会创建“类对应的散列表”
这里所说的“不会创建类对应的散列表”是说：我们不会在HashSet, Hashtable, HashMap等等这些本质是散列表的数据结构中，用到该类。例如，不会创建该类的HashSet集合。
在这种情况下，该类的“hashCode() 和 equals() ”没有半毛钱关系的！
这种情况下，equals() 用来比较该类的两个对象是否相等。而hashCode() 则根本没有任何作用，所以，不用理会hashCode()。
第二种 会创建“类对应的散列表”
这里所说的“会创建类对应的散列表”是说：我们会在HashSet, Hashtable, HashMap等等这些本质是散列表的数据结构中，用到该类。例如，会创建该类的HashSet集合。
在这种情况下，该类的“hashCode() 和 equals() ”是有关系的：
1)、如果两个对象相等，那么它们的hashCode()值一定相同。
    这里的相等是指，通过equals()比较两个对象时返回true。
2)、如果两个对象hashCode()相等，它们并不一定相等。
     因为在散列表中，hashCode()相等，即两个键值对的哈希值相等。然而哈希值相等，并不一定能得出键值对相等。补充说一句：“两个不同的键值对，哈希值相等”，这就是哈希冲突。
此外，在这种情况下。若要判断两个对象是否相等，除了要覆盖equals()之外，也要覆盖hashCode()函数。否则，equals()无效。
http://www.cnblogs.com/skywang12345/p/3324958.htm

# 接口与抽象类的区别
抽象类：抽象类是用来捕捉子类的通用特性的 。它不能被实例化，只能被用作子类的超类。抽象类是被用来创建继承层级里子类的模板
接口：接口是抽象方法的集合。如果一个类实现了某个接口，那么它就继承了这个接口的抽象方法。这就像契约模式，如果实现了这个接口，那么就必须确保使用这些方法。接口只是一种形式，接口自身不能做任何事情。
![接口与抽象类的区别](http://7xilc8.com1.z0.glb.clouddn.com/interfaceabstrct.png)

如果你拥有一些方法并且想让它们中的一些有默认实现，那么使用抽象类。
如果你想声明一些非公共的成员，使用抽象类，因为接口中所有的方法都是public。
如果你想实现多重继承，那么你必须使用接口。由于Java不支持多继承，子类不能够继承多个类，但可以实现多个接口。因此你就可以使用接口来解决它。
如果基本功能在不断改变，那么就需要使用抽象类。
如果不断改变基本功能并且使用接口，那么就需要改变所有实现了该接口的类。
http://www.importnew.com/12399.html
http://www.programmerinterview.com/index.php/java-questions/interface-vs-abstract-class/
# java三大特性
面向对象编程有三大特性：**封装、继承、多态**。

封装隐藏了类的内部实现机制，可以在不影响使用的情况下改变类的内部结构，同时也保护了数据。对外界而已它的内部细节是隐藏的，暴露给外界的只是它的访问方法。

继承是为了重用父类代码。两个类若存在IS-A的关系就可以使用继承。同时继承也为实现多态做了铺垫。

所谓多态就是指程序中定义的引用变量所指向的具体类型和通过该引用变量发出的方法调用在编程时并不确定，而是在程序运行期间才确定，即一个引用变量倒底会指向哪个类的实例对象，该引用变量发出的方法调用到底是哪个类中实现的方法，必须在由程序运行期间才能决定。因为在程序运行时才确定具体的类，这样，不用修改源程序代码，就可以让引用变量绑定到各种不同的类实现上，从而导致该引用调用的具体方法随之改变，即不修改程序代码就可以改变程序运行时所绑定的具体代码，让程序可以选择多个运行状态，这就是多态性。静态单分派，动态多分派

http://www.bkjia.com/Javabc/1112062.html

# springMVC注解
http://www.cnblogs.com/leskang/p/5445698.html

https://wapwenku.baidu.com/view/ebbb24558f9951e79b89680203d8ce2f006665ba.html?pn=7&vw=all&pu=usm@1,sz@320_1001,ta@iphone_2_7.0_3_537
