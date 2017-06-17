title: IO相关
toc: true
date: 2017-03-17 21:33:44
tags: java
categories: note
---
java.io中可以分为4个大类
传输数据的格式可以分为字节和字符
基于**字节**的I/O接口:InputStream和OutputSream
基于**字符**的I/O接口:Writer和Reader
传输数据的方式可以分为磁盘和网络
基于**磁盘**的I/O接口:File
基于**网络**的I/O接口:Socket(不是io包)

不管是网络还是磁盘，最小的存储单元仍然是**字节**而不是字符。
而程序中经常是以字符为单位的，所以还是要有字符的接口的。
http://www.jianshu.com/p/9fe3b51cf055
#  访问文件的方式
## 标准的文件访问
![标准的文件访问](http://7xilc8.com1.z0.glb.clouddn.com/io1.jpg)
## 直接的io方式
直接就是不经过操作系统的内核数据缓冲区，直接访问磁盘数据，这样做的目的就是为了减少一次从内核缓冲区到用户缓存的数据复制。
比如数据库管理系统。但是如果数据不在应用程序缓存中，每次从磁盘直接加载，就会很慢，所以直接io通常与异步io相结合。
## 同步/异步访问文件
同步：只有数据被成功写入文件，才会返回成功给应用程序。
异步：当访问数据的线程发出请求后，线程会接着处理其他事情，而不会阻塞，当请求的数据返回时才会继续处理下面的操作。
## 内存映射
指的是操作系统将内存中的某一块区域与磁盘的文件关联起来，当要访问内存中一段数据时，转换为访问文件中国的一段数据，做到空间共享。
http://www.jianshu.com/p/968d2c2d4471
# 编码
## 编码分类
### 字符集和字符编码
1、字符集（charset）
字符集指的是一个系统支持的所有抽象字符的集合。字符是各种文字和符号的总称，包括各国家文字、标点符号、图形符号、数字等，常见的字符集有ASCII字符集、GB2312字符集、BIG5字符集、GB18030字符集、Unicode字符集等。
2、字符编码（encoding）
计算机要准确处理各种字符集文字，就要进行字符编码，以便计算机能够识别和存储各种文字。因此字符编码就是讲符号转换为计算机可以接受的数字系统的数，称为数字代码。

### ASCII码

计算机里面只有数字0和1（严格说连0和1都没有，只有开和关，无非是用0和1表示开关的状态罢了），在计算机软件里的一切都是用数字标识的额，屏幕上显示的一个一个字符也是数字。最初使用的计算机在美国，用到的字符很少，因此每一个字符都用一个数字表示，一个字节所能表示的数字反内卫足以容纳所有这些字符。
实际上表示这些字符的数字的字节最高位都是0，也就是说这些数字都在0~127之间，如字符a对应97，字符b对应数字98，这种字符与数字的对应编码固定下来之后，这套编码规则被称为ASCII码（美国标准信息交换码）。

1、0~31是控制字符，如换行、回车、删除等
2、32~126是打印字符，可以通过键盘输入并且能够显示出来

### GB2312和GBK

随着计算机在其它国家的普及，许多国家把本地字符集引入了计算机，大大扩展了计算机中字符的范围。
一个字节所能表示的范围不足以容纳中文字符（看看上面的ASCII码表就知道了），中国大陆将每一个中文字符都用两个字节表示，原有的ASCII码字符的编码保持不变。

为了将一个中文字符与两个ASCII码字符相区别，中文字符的每个字节最高位为1，中国大陆为每一个中文字符都指定了一个对应的数字，并于1980年制定了一套《信息技术 中文编码字符集》，这套规范就是GB2312。GB2312是双字节编码，总的编码范围是A1~F7，其实A1~A9是富豪区，总共包含682个符号；B0~F7是汉字区，总共包含6763个汉字。

GBK是在1995年制定的后续标准，全称为《汉字内码扩展规范》，是国家技术监督局为Windows 95所制定的新的汉字内码规范。
**GBK的出现是为了扩展GBK2312，并加入更多的汉字**。GBK的编码范围是8140~FEFE（去掉XX7F），总共有23940个码位，能表示21003个汉字，它的编码是和GB2312兼容的，也就是说用GB2312编码的汉字可以用GBK来解码，并且不会有乱码问题。GBK还是现如今中文Windows操作系统的系统默认编码。

### Unicode

在一个国家的本地化系统中出现的一个字符，通过电子邮件传送到另外一个国家的本地化系统中，看到的就不是那个原始字符了，而是另外那个国家的一个字符或乱码，因为计算机里面并没有真正的字符，字符都是以数字的形式存在的，通过邮件传送一个字符，实际上传送的是这个字符对应的字符编码，同一个数字在不同的国家和地区代表的很可能是不同的符号。

为了解决各个国家和地区之间各自使用不同的本地化字符编码带来的不便，人们将全世界所有的符号进行了统一编码，称之为Unicode（统一码、万国码）。
所有字符不再区分国家和地区，都是人类共有的符号，如"中"字在Unicode中不再是GBK中的D6D0，而是在任何地方都是4e2d，如果所有的计算机系统都使用这种编码方式，那么4e2d这个字在任何地方都代表汉字中的"中"。Unicode编码的字符都占用两个字节的大小，也就是说全世界所有字符个数不会超过65536个。

当然Unicode只包含65536个字符就想包含全世界所有的字符是远远不够的，所以Unicode提供了字符平面映射。
另外要提一点的是，Unicode是Java和XML的基础。


UTF-8和UTF-16

Unicode是一种字符集标准，而具体该标准应该如何应用到计算机中，则是另一个话题了，常用的Unicode编码方式有两种：

1、UTF-16。两个字节表示Unicode转换格式，这是定长的表示方法。
**也就是说不管什么字符都可以使用两个字节表示，两个字节是16Bit，所以叫做UTF-16**。UTF-16编码非常方便，每两个字节表示一个字符，这个在字符串操作时大大简化了操作。

2、UTF-8。UTF-16统一采用了两个字节表示一个字符，虽然在表示上非常简单，但是很大一部分字符用一个字节表示就够了，现在需要两个字节，存储空间放大了一倍。
**UTF-8就采取了一种变长技术，每个编码区域有不同的字码长度，不同类型的字符可以是由1~6个字节组成。**

两种编码方式比较，相对来说，UTF-16的编码效率较高，从字符到字节的相互转换可以更简单，进行字符串操作也更好，它更适合在本地磁盘和内存之间使用，可以进行字符和字节之间的快速切换。
但是UTF-16并不适合在网络之间传输，因为网络传输易损坏字节流，一旦字节流损坏将很难恢复，所以相比较而言**UTF-8更适合网络传输**。
另外UTF-8对ASCII字符采用单字节存储，单个字符损坏也不会影响后面的其他字符，在编码效率上介于GBK和UTF-16之间，所以，UTF-8在编码效率和编码安全性上做了平衡，是理想的中文编码方式。

**Java中的字符使用的都是Unicode字符集，编码方式为UTF-16**

## 转化桥梁
数据持久化或者网络传输中都是以**字节**的进行的，因此有必要完成从字符到字节的转化。
![标准的文件访问](http://7xilc8.com1.z0.glb.clouddn.com/inputstreamreader.jpg)

`InputStreamReader`就是`Reader`和`InputStream`的转化桥梁

```
public class InputStreamReader extends Reader {
    private final StreamDecoder sd;
    ...
}
public class StreamDecoder extends Reader {
    private Charset cs;
    private CharsetDecoder decoder;
    private InputStream in;
    ...
}
```
这也是适配器模式的一种体现。

# 序列化
序列化就是将一个对象转化为一串二进制表示的字节数组，通过保存或转移这些字节数据来达到持久化(Java平台允许我们在内存中创建可复用的Java对象，但一般情况下，只有当JVM处于运行时，这些对象才可能存在，即，这些对象的生命周期不会比JVM的生命周期更长。但在现实应用中，就可能要求在JVM停止运行之后能够保存(持久化)指定的对象，并在将来重新读取被保存的对象。Java对象序列化就能够帮助我们实现该功能。)的目的。
如果要持久化必须要继承`java.io.Serializable`。

如果仅仅只是让某个类实现Serializable接口，而没有其它任何处理的话，则就是使用默认序列化机制。
使用默认机制，在序列化对象时，不仅会序列化当前对象本身，还会对该对象引用的其它对象也进行序列化，同样地，这些其它对象引用的另外对象也将被序列化，以此类推。
所以，如果一个对象包含的成员变量是容器类对象，而这些容器所含有的元素也是容器类对象，那么这个序列化的过程就会较复杂，开销也较大。
在纯java的情况下，序列化能很好地工作，但是多语言环境中，用java序列化存储之后。很难用其他语言还原回来。
因此通常使用json或xml结构数据，或者比较好的序列化工具如protobuf

**序列化并不保存静态变量**

## 序列化方法
假定一个Student类，它的对象需要序列化，可以有如下三种方法：

方法一：
若Student类仅仅实现了Serializable接口，则可以按照以下方式进行序列化和反序列化
ObjectOutputStream采用默认的序列化方式，对Student对象的**非transient**的实例变量进行序列化。
ObjcetInputStream采用默认的反序列化方式，对对Student对象的**非transient**的实例变量进行反序列化。

方法二：
若Student类仅仅实现了Serializable接口，并且还定义了readObject(ObjectInputStream in)和writeObject(ObjectOutputSteam out)，则采用以下方式进行序列化与反序列化。
ObjectOutputStream调用Student对象的writeObject(ObjectOutputStream out)的方法进行序列化。
ObjectInputStream会调用Student对象的readObject(ObjectInputStream in)的方法进行反序列化。

方法三：若Student类实现了Externalnalizable接口，且Student类必须实现readExternal(ObjectInput in)和writeExternal(ObjectOutput out)方法，则按照以下方式进行序列化与反序列化。
ObjectOutputStream调用Student对象的writeExternal(ObjectOutput out))的方法进行序列化。
ObjectInputStream会调用Student对象的readExternal(ObjectInput in)的方法进行反序列化。

方法二和方法三可以实现特殊字段的特殊系列化方法
### JDK类库中序列化和反序列化的步骤

- JDK类库中序列化
步骤一：创建一个对象输出流，它可以包装一个其它类型的目标输出流，如文件输出流：
`ObjectOutputStream out = new ObjectOutputStream(new fileOutputStream(“D:\\objectfile.obj”));`
步骤二：通过对象输出流的writeObject()方法写对象：
`out.writeObject(“Hello”);
out.writeObject(new Date());`

- JDK类库中反序列化的步骤：
步骤一：创建一个对象输入流，它可以包装一个其它类型输入流，如文件输入流：
`ObjectInputStream in = new ObjectInputStream(new fileInputStream(“D:\\objectfile.obj”));`
步骤二：通过对象输出流的readObject()方法读取对象：
`String obj1 = (String)in.readObject();
Date obj2 = (Date)in.readObject();`

说明：为了正确读取数据，完成反序列化，必须保证向对象输出流写对象的顺序与从对象输入流中读对象的顺序一致。
http://developer.51cto.com/art/201202/317181.htm
https://www.ibm.com/developerworks/cn/java/j-lo-serial/

### 序列化相关问题
#### 如何保证对象的引用关系
比如
```java
User u = new User("001");
Order o1 = new Order(1,u);
Order o2 = new Order(2,u);
```
只有**相同的ObjectOutputStream**才能保证反序列化出来的两个order中的user是相同的
另外，序列化和反序列化可以在不同的工程里面，但是前提是，**反序列化的工程中必须有反序列化对象的class文件**。

#### serialVersionId的作用
可以保证class文件的兼容性，也就是说，即使序列化和反序列化中的对象属性不完全相同(序列化中的user只有name属性，反序列化中的user有name和age属性)，只要serialVersionId相同，就可以序列化成功
但是，**反序列化过程不会调用构造函数，同时变化的属性值为默认值**，也就是说对于新的属性，使用构造函数初始化值，在反序列化过程中是无效的

# IO简单总结
总结一下流类的使用

1、File是一些文件/文件夹操作的源头，File代表的就是文件/文件夹本身，因此无论如何，使用IO的第一步是建议开发者根据路径实例化出一个File
2、考虑使用字符流还是字节流。操作文本一般使用字符流，即Reader和Writer；操作字节文件使用字节流，即InputStream和OutputStream
3、选择使用输入流还是输出流。把内容从文件读入Java内存使用输入流，即Reader和InputStream；把内容从Java内存读到文件使用输出流，即Writer和OutputStream
4、使用字符流使用BufferedReader和BufferedWriter，它们的构造函数中的参数分别是Reader和Writer，因此既可以实例化出FileReader和FileWriter，也可以实例化出InputStreamReader和OutputStreamWriter，作为构造函数的参数传入BufferedReader和BufferedWriter
5、FileInputStream和FileOutputStream可以直接操作文件的读写，它们没有做缓存
6、ObjectOutputStream和ObjectInputStream，它们分别以OutputStream和InputStream作为构造函数的参数，因此可以实例化出FileOutputStream和FileInputStream并传入
![Reader](http://7xilc8.com1.z0.glb.clouddn.com/reader.png)
![Writer](http://7xilc8.com1.z0.glb.clouddn.com/writer.png)
![InputStream](http://7xilc8.com1.z0.glb.clouddn.com/inputstream.png)
![OutputStream](http://7xilc8.com1.z0.glb.clouddn.com/outputstream.png)

https://www.ibm.com/developerworks/cn/java/j-lo-javaio/



# 设计模式
## 适配器
将一个类的接口转换成客户希望的另外一个接口。Adapter 模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作。

Java I/O 库大量使用了适配器模式，例如 ByteArrayInputStream 是一个适配器类，它继承了 InputStream 的接口，并且封装了一个 byte 数组。
换言之，它将一个 byte 数组的接口适配成 InputStream 流处理器的接口。
我们知道 Java 语言支持四种类型：Java 接口，Java 类，Java 数组，原始类型（即 int,float 等）。
前三种是引用类型，类和数组的实例是对象，原始类型的值不是对象。也即，Java 语言的数组是像所有的其他对象一样的对象，而不管数组中所存储的元素类型是什么。
这样一来的话，ByteArrayInputStream 就符合适配器模式的描述，是一个对象形式的适配器类。

FileInputStream 是一个适配器类。在 FileInputStream 继承了 InputStrem 类型，同时持有一个对 FileDiscriptor 的引用。这是将一个 FileDiscriptor 对象适配成 InputStrem 类型的对象形式的适配器模式。FileOutputStream同理
```java
public class FileInputStream extends InputStream
{
    /* File Descriptor - handle to the open file */
    private final FileDescriptor fd;
    ...
}
```
同样地，在 OutputStream 类型中，所有的原始流处理器都是适配器类。
ByteArrayOutputStream 继承了 OutputStream 类型，同时持有一个对 byte 数组的引用。
它一个 byte 数组的接口适配成 OutputString 类型的接口，因此也是一个对象形式的适配器模式的应用。

Reader 类型的原始流处理器都是适配器模式的应用。
StringReader 是一个适配器类，StringReader 类继承了 Reader 类型，持有一个对 String 对象的引用。它将 String 的接口适配成 Reader 类型的接口。


https://www.ibm.com/developerworks/cn/java/j-lo-adapter-pattern/#ibm-pcon

## 装饰器
装饰器模式又称为包装（Wrapper）模式。装饰器模式以多客户端透明的方式扩展对象的功能，是继承关系的一个替代方案。

### 装饰器模式的结构

通常给对象添加功能，要么直接修改对象添加相应的功能，要么派生子类来扩展，抑或是使用对象组合的方式。
显然，直接修改对应的类的方式并不可取，在面向对象的设计中，我们应该尽量使用组合对象而不是继承对象来扩展和复用功能，装饰器模式就是基于对象组合的方式的。
装饰器模式以对客户端透明的方式动态地给一个对象附加上了更多的责任。
换言之，客户端并不会角色对象在装饰前和装饰后有什么不同。装饰器模式可以在不用创建更多子类的情况下，将对象的功能加以扩展。

### 装饰器在io中的分析
![io类中装饰器模式分析](http://7xilc8.com1.z0.glb.clouddn.com/iodecorator.jpg)

有了上面的图片，在使用基本io时就好办多了。
首先根据需求来判断是操作字节（InputStream 和 OutputStream），还是操作字符（Reader 和 Writer）来选择不同的分支。
然后根据是操作对象还是文件等来选择具体的组件（装饰者模式的 ConcreteComponent），最后根据是否需要缓冲区等功能来使用相应的装饰者类来包装具体组件类。

简单例子：以字节流方式读取图片。因为读取的是图片，因此不能选择字符操作的 Reader，应选择 InputStream；因为图片属于文件系统中的文件，所以选择FileInputStream；又因为我们觉得一次读取一个字节不合适，我们又使用 BufferedInputStream 来包装具体组件 FileInputStream。因此有了以下代码
`InputStream ips = new BufferedInputStream( new FileInputStream("..."))`

我们这里实例化出了三个InputStream的实现类：

```
File file = new File("D:/aaa.txt");
FileInputStream in0 = new FileInputStream(file);
BufferedInputStream in1 = new BufferedInputStream(new FileInputStream(file));
DataInputStream in2 = new DataInputStream(new BufferedInputStream(new FileInputStream(file)));
```
1、in0这个引用指向的是new出来的FileInputStream，这和装饰器模式无关，因为InputStream本身就是一个输入流，FileInputStream实现了最基本的功能罢了
2、in1这个引用指向的是new出来的BufferedInputStream，这就和装饰器模式有关了，它给FileInputStream增加了功能，使得FileInputStream读取文件的内容保存在内存中，以提高读取的功能
3、in2这个引用指向的是new出来的DataInputStream，这也和装饰器模式有关，因为它也给FileInputStream增加了功能，因为它有DataInputStream和BufferedInputStream两个附加的功能
2和3这两点，对于客户端来说，只是使用InputStream的方法而已，根本不知道某个InputStream是否被装饰了，如果被装饰了，InputStream在装饰前和装饰后有什么区别。在客户端不知情的情况下，给这些方法附加上了额外的功能，这就是装饰器模式。

### 装饰器模式的优缺点

优点

1、装饰器模式与继承关系的目的都是要扩展对象的功能，但是装饰器模式可以提供比继承更多的灵活性。
装饰器模式允许系统动态决定贴上一个需要的装饰，或者除掉一个不需要的装饰。继承关系是不同，继承关系是静态的，它在系统运行前就决定了
2、通过使用不同的具体装饰器以及这些装饰类的排列组合，设计师可以创造出很多不同的行为组合

缺点

由于使用装饰器模式，可以比使用继承关系需要较少数目的类。使用较少的类，当然使设计比较易于进行。
但是另一方面，由于使用装饰器模式会产生比使用继承关系更多的对象，更多的对象会使得查错变得困难，特别是这些对象看上去都很像。



## 装饰器模式和适配器模式的区别

其实适配器模式也是一种包装（Wrapper）模式，它们看似都是起到包装一个类或对象的作用，但是它们使用的目的非常不一样：

1、适配器模式的意义是要**将一个接口转变成另外一个接口**，它的目的是通过改变接口来达到重复使用的目的
2、装饰器模式不要改变被装饰对象的接口，而是恰恰要**保持原有的接口，但是增强原有接口的功能**，或者改变元有对象的处理方法而提升性能

所以这两种设计模式的目的是不同的。
