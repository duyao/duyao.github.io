title: String相关
toc: true
date: 2017-03-12 15:23:56
tags: java
categories: note
---
# String

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
}
```
1）String类是**final类**，也即意味着String类不能被继承，并且它的成员方法都默认为final方法。
2）上面列举出了String类中所有的成员属性，从上面可以看出String类其实是通过**char数组来保存字符串**的

## 为什么是string是不可变的

安全性：网络连接中的参数通常是string的数据库url、password

Security: parameters are typically represented as String in network connections, database connection urls, usernames/passwords etc. If it were mutable, these parameters could be easily changed.
Synchronization and concurrency: making String immutable automatically makes them thread safe thereby solving the synchronization issues.
Caching: when compiler optimizes your String objects, it sees that if two objects have same value (a="test", and b="test") and thus you need only one string object (for both a and b, these two will point to the same object).
Class loading: String is used as arguments for class loading. If mutable, it could result in wrong class being loaded (because mutable objects change their state).
http://stackoverflow.com/questions/22397861/why-is-string-immutable-in-java

## hashCode 和 equal方法
 hash code(散列码，也可以叫哈希码值)是对象产生的一个整型值。
 其生成没有规律的。二者散列码可以获取对象中的信息，转成那个对象的“相对唯一”的整型值。所有对象都有一个散列码。
`h = 31 * h + val[i];`这里的计算为什么要用31呢？看看《Effective Java》第二版第三章第42页中的原话：
之所以选择31，是因为它是个奇素数，如果乘数是偶数，并且乘法溢出的话，信息就会丢失，因为与2相乘等价于移位运算。使用素数的好处并不是很明显，但是习惯上都使用素数来计算散列结果。
31有个很好的特性，就是用移位和减法来代替乘法，可以得到更好的性能：31*i==(i<<5)-i。现在的VM可以自动完成这种优化。

```java
public int hashCode() {
    int h = hash;
    if (h == 0 && value.length > 0) {
        char val[] = value;

        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
```
如果传入参数地址相同，那一定是相等的。
如果传入参数anObject是String类型，那么就循环迭代这两个字符串对应的字符数组中的每一个值是否相等，如有一个字符不相等，则返回false。
```java
public boolean equals(Object anObject) {
    if (this == anObject) {
        return true;
    }
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = value.length;
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            while (n-- != 0) {
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
```

## hash常见方法

XXX.hashCode(primitive); // primitve 是基本类型，XXX 是 primitve 的 Wrapper Class
Objects.hashCode(obj); // obj 是对象
Arrays.hashCode(ary); // ary 是各种数组
Objects.hash(x, y, z); // 参数是 varargs，可以是多个各种类型，但不建议参数个数只有一个

## 重写equals
根据《effective java》这本书的内容，进行总结一下：

我们若要重写equals方法时，必须遵守他的通用约定，否则会出现异常。

- 对称性（symmetric）：任何非空=null引用值x y，如果x.equals(y)返回是“true”，那么y.equals(x)也应该返回是“true”。
- 自反性（reflexive）：任何非空=null引用值 x，x.equals(x)必须返回是“true”。
- 传递性（transitive）：任何非空=null引用值x y z ，如果x.equals(y)返回是“true”，而且y.equals(z)返回是“true”，那么z.equals(x)也应该返回是“true”。
- 一致性（consistent）：任何非空=null引用值x y,如果x.equals(y)返回是“true”，只要x和y内容一直不变，不管你重复x.equals(y)多少次，返回都是“true”。
- 任何情况下，**x.equals(null)，永远返回是“false”**；**x.equals(和x不同类型的对象)永远返回false**

当我们重写equals方法时，也要重写hashCode方法，因为我们要保证，如果两个对象equals等于true，那么hashCode返回值必须也是相等的
1、当两个对象equals方法返回true，那么hashCode返回值一定是相等的；如果两个对象equals方法返回值是false，那么这两个对象的hashCode的返回值有可能相等。
2、当两个对象的hashCode返回值相等时，两个对象的equals方法不一定true（hash冲突）；但是如果**两个对象的hashCode返回值不相等，那么这个对象equals返回值一定是false**.

## 其他方法
```java
public String substring(int beginIndex, int endIndex) {
    if (beginIndex < 0) {
        throw new StringIndexOutOfBoundsException(beginIndex);
    }
    if (endIndex > value.length) {
        throw new StringIndexOutOfBoundsException(endIndex);
    }
    int subLen = endIndex - beginIndex;
    if (subLen < 0) {
        throw new StringIndexOutOfBoundsException(subLen);
    }
    //当子串还是原来的就不会new，而是返回原来的
    return ((beginIndex == 0) && (endIndex == value.length)) ? this
            : new String(value, beginIndex, subLen);
}

public String concat(String str) {
  int otherLen = str.length();
  if (otherLen == 0) {
      return this;
  }
  char buf[] = new char[count + otherLen];
  getChars(0, count, buf, 0);
  str.getChars(0, otherLen, buf, count);
  return new String(0, count + otherLen, buf);
}
```
可以看出，string 的方法都是返回new出来的新字符串，只有当结果不变的时候才是返回原字符串的
**“对String对象的任何改变都不影响到原对象，相关的任何change操作都会生成新的对象”。**

## 一个例子
先看java内存区域
![ ](http://7xilc8.com1.z0.glb.clouddn.com/s1.png)
执行`String str = "hello"`语句的过程
首先编译，形成class文件
然后进入加载阶段：把class文件加载到虚拟机中，class文件中大部分的数据会被加载到“运行常量池running time pool”中
但是string不会，**其中的“hello”的一个引用会被存在“字符串常量池中string pool”，而“hello”本体还是和所有对象一样创建在堆heap上**
`这只是在Test类被类加载器加载时候的情形。主线程中的str变量这时候都还没有被创建，但Hello的实例已经在Heap里了，对它的引用也已经在字符串常量池里了。`
![ ](http://7xilc8.com1.z0.glb.clouddn.com/s2.png)
等主线程开始创建str变量的时候，虚拟机就会到字符串常量池里找，看有没有能equals("Hello")的String。
如果找到了，就在栈区当前栈帧的局部变量表里创建str变量，然后把字符串常量池里对Hello对象的引用复制给str变量。
找不到的话，才会在heap堆重新创建一个对象，然后把引用驻留到字符串常量区。然后再把引用复制栈帧的局部变量表。
![ ](http://7xilc8.com1.z0.glb.clouddn.com/s3.png)


如果我们当时定义了很多个值为"Hello"的String，比如`String str1 = "hello";String str2 = "hello";String str3 = "hello";`，有三个变量str1,str2,str3，也不会在堆上增加String实例。局部变量表里三个变量统一指向同一个堆内存地址。
![ ](http://7xilc8.com1.z0.glb.clouddn.com/s4.png)

但如果是用new关键字来创建字符串` String str1 = "Hello";String str2 = "Hello";String str3 = new String("Hello");`，情况就不一样了
![ ](http://7xilc8.com1.z0.glb.clouddn.com/s5.png)
这时候，str1和str2还是和之前一样。但str3因为new关键字会在Heap堆申请一块全新的内存，来创建新对象。虽然字面还是"Hello"，但是完全不同的对象，有不同的内存地址。

https://www.zhihu.com/question/29884421
关于不同的常量池http://tangxman.github.io/2015/07/27/the-difference-of-java-string-pool/

# StringBuffer和StringBuilder
初始大小都是16
而StringBuffer是线程安全的，因为他的方法被`synchronized`修饰，而StringBuilder不安全
所以通常来说**StringBuilder > StringBuffer > String**

```java
//StringBuffer
public synchronized String toString() {
    if (toStringCache == null) {
        toStringCache = Arrays.copyOfRange(value, 0, count);
    }
    return new String(toStringCache, true);
}

//StringBuilder
public String toString() {
    // Create a copy, don't share the array
    return new String(value, 0, count);
}
```

## 与String的关系
```java
public static void main1(String[] args) {
    String string = "";
    for(int i=0;i<10000;i++){
        string += "hello";
    }
}

public static void main2(String[] args) {
    StringBuilder stringBuilder = new StringBuilder();
    for(int i=0;i<10000;i++){
        stringBuilder.append("hello");
    }
}
```
在main1中，每次循环会**new出一个StringBuilder对象，然后进行append操作，最后通过toString方法返回String对象。**
也就是说这个循环执行完毕new出了10000个对象，试想一下，如果这些对象没有被回收，会造成多大的内存资源浪费。
在main2中，new操作只进行了一次，也就是说只生成了一个对象，append操作是在原有对象的基础上进行的。因此在循环了10000次之后，这段代码所占的资源要比上面小得多。


```java
public static void test1String () {
   long begin = System.currentTimeMillis();
   for(int i=0; i<time; i++){
       String s = "I"+"love"+"java";
   }
   long over = System.currentTimeMillis();
   System.out.println("字符串直接相加操作："+(over-begin)+"毫秒");
}

public static void test2String () {
   String s1 ="I";
   String s2 = "love";
   String s3 = "java";
   long begin = System.currentTimeMillis();
   for(int i=0; i<time; i++){
       String s = s1+s2+s3;
   }
   long over = System.currentTimeMillis();
   System.out.println("字符串间接相加操作："+(over-begin)+"毫秒");
}
```
由此可以得出结论：
1）对于直接相加字符串，效率很高，因为在编译器便确定了它的值，也就是说形如`"I"+"love"+"java"`; 的字符串相加，在编译期间便被优化成了`"Ilovejava"`。这个可以用javap -c命令反编译生成的class文件进行验证。
2）对于间接相加（即包含字符串引用），形如s1+s2+s3; 效率要比直接相加低，因为在编译器不会对引用变量进行优化。


# 一些例子
1）下面这段代码的输出结果是什么？

`String a = "hello2"; 　　String b = "hello" + 2; 　　System.out.println((a == b));`

输出结果为：true。原因很简单，"hello"+2在编译期间就已经被优化成"hello2"，因此在运行期间，变量a和变量b指向的是同一个对象。

2）下面这段代码的输出结果是什么？

`String a = "hello2"; 　  String b = "hello";       String c = b + 2;       System.out.println((a == c));`

输出结果为:false。由于有符号引用的存在，所以  String c = b + 2;不会在编译期间被优化，不会把b+2当做字面常量来处理的，因此这种方式生成的对象事实上是保存在堆上的。因此a和c指向的并不是同一个对象。

3）下面这段代码的输出结果是什么？

`String a = "hello2";   　 final String b = "hello";       String c = b + 2;       System.out.println((a == c));`

输出结果为：true。对于被final修饰的变量，会在class文件常量池中保存一个副本，也就是说不会通过连接而进行访问，对final变量的访问在编译期间都会直接被替代为真实的值。那么String c = b + 2;在编译期间就会被优化成：String c = "hello" + 2;


4）下面这段代码输出结果为：
```java
public class Main {
    public static void main(String[] args) {
        String a = "hello2";
        final String b = getHello();
        String c = b + 2;
        System.out.println((a == c));
    }

    public static String getHello() {
        return "hello";
    }
}
```
输出结果为false。这里面虽然将b用final修饰了，但是由于其赋值是通过方法调用返回的，那么它的值只能在运行期间确定，因此a和c指向的不是同一个对象。

5）下面这段代码的输出结果是什么？

```
public class Main {
    public static void main(String[] args) {
        String a = "hello";
        String b =  new String("hello");
        String c =  new String("hello");
        String d = b.intern();

        System.out.println(a==b);
        System.out.println(b==c);
        System.out.println(b==d);
        System.out.println(a==d);
    }
}
```

ffft
这里面涉及到的是String.intern方法的使用。在String类中，intern方法是一个本地方法，在JAVA SE6之前，intern方法会在运行时常量池中查找是否存在内容相同的字符串，如果存在则返回指向该字符串的引用，如果不存在，则会将该字符串入池，并返回一个指向该字符串的引用。因此，a和d指向的是同一个对象。
参考书57页

6）下面这段代码1）和2）的区别是什么？
```
public class Main {
    public static void main(String[] args) {
        String str1 = "I";
        //str1 += "love"+"java";        1)
        str1 = str1+"love"+"java";      //2)

    }
}
```
1）的效率比2）的效率要高，1）中的"love"+"java"在编译期间会被优化成"lovejava"，而2）中的不会被优化。
可以看出，在1）中只进行了一次append操作，而在2）中进行了两次append操作。




堆（Heap）：最大一块空间。存放对象实例和数组。全局共享

http://rednaxelafx.iteye.com/blog/774673/
http://www.cnblogs.com/dolphin0520/p/3778589.html
