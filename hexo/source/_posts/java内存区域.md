title: java内存区域
toc: true
date: 2017-02-22 09:31:32
tags: jvm
categories: note
---
# 简介
在一台物理机中，内存可以分为物理内存和虚拟内存(Linux中swap的空间活跃度反应着物理内存)
而对于内存的使用是由该机器的操作系统完成，因此有可以划分为内核空间和用户空间。
在java中，需要向操作系统申请内存的主要有：java堆、线程、类和类加载器、NIO、JNI(native memory)
# java内存分配

## java虚拟机运行时数据区

主要分为5个部分：方法区(常量池)、堆、虚拟机栈、本地方法栈和程序计数器
隔离vpn，方法堆共享(vpn指的是v虚拟机栈virtual machine stack,p指的是program counter register程序计数器，n指的是本地方法栈native methid stack)

- 程序计数器Program Counter Register
不共享，用来记录当前线程所执行的字节码行号，**每个线程都有自己的程序计数器**

- java虚拟机栈 Java Virtual Machine Stack
线程私有，与线程生命周期相同，**存放基本类型和对象引用**
其内部存放java方法的内存模型，**每一个线程的执行都会创建一个栈，而线程内部每个方法执行的时候都会创建一个栈帧stack frame**，方法完成时就会将该栈帧移除。
**栈帧中有局部变量表、操作数栈**、动态链接、方法返回地址和一些额外的附加信息。在**编译**程序代码时，栈帧中需要多大的局部变量表、多深的操作数栈都已经**完全确定**了，并且写入了方法表的Code属性之中。
因此，一个栈帧需要分配多少内存，不会受到程序运行期变量数据的影响，而仅仅取决于具体的虚拟机实现。
栈帧的入栈出栈过程对应着方法的执行与完成


- 本地方法栈 Native Method Stack
java虚拟机栈为java方法（字节码）服务，本地方法栈为虚拟机使用到的Native方法服务

- Java堆 Heap
共享，存放着**对象实例和数组**，都是非静态属性，是垃圾回收的主要区域
所有的对象实例都会在堆上分配。这句话随着逃逸分析技术的成熟变得不是那么绝对了。
逃逸分析就是分析一个实例是否会被其他对象拥有甚至改变，如果没有，说明这个对象是私有的，那么为了减小gc压力，完全可以放在堆上分配。
java堆可以划分为新生代和老年代，新生代还可以划分为eden、from survivor、to survivor。
java堆是物理上不连续的内存空间，只要是逻辑上连续即可，就像磁盘空间。

- 方法区Method Area
共享，用于存放class的相关信息，**存放“类”被加载后的信息，常量，静态变量**
其中包含**运行时常量池**，具有动态性，运行期间也可能有新的常量放入池中，比如String类的intern()方法和大量产生class文件的应用，比如CGLib字节码增强技术，Jsp文件的应用(Jsp第一次运行时需要便以为java类)、基于OSGI的应用

- 直接内存
javaNIO通过通道与缓冲区的方法分配使用

## 对象的创建过程




1、 遇到new指令，先去常量池定位这个类的引用，如果没有加载过，就执行类的加载过程
2、 类加载检查完成后，为新生对象分配内存，这个大小是在加载过程中就可以确定的
3、 虚拟机对对象进行必要的设置，即gc分代年龄、锁、元数据等信息放在对象头中。
HotSpot虚拟机中，对象在内存中存储的布局可以分为三块区域：对象头（Header）、实例数据（Instance Data）和对齐填充（Padding）。  

HotSpot虚拟机的对象头(Object Header)包括两部分信息：
第一部分用于存储对象自身的运行时数据， 如哈希码（HashCode）、GC分代年龄、锁状态标志、线程持有的锁、偏向线程ID、偏向时间戳等等，这部分数据的长度在32位和64位的虚拟机（暂 不考虑开启压缩指针的场景）中分别为32个和64个Bits，官方称它为“Mark Word”。
对象头的另外一部分是类型指针，即是对象指向它的类的元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例。
并不是所有的虚拟机实现都必须在对象数据上保留类型指针，换句话说查找对象的元数据信息并不一定要经过对象本身。
另外，如果对象是一个Java数组，那在对象头中还必须有一块用于记录数组长度的数据，因为虚拟机可以通过普通Java对象的元数据信息确定Java对象的大小，但是从数组的元数据中无法确定数组的大小。  
![堆中的对象头](http://7xilc8.com1.z0.glb.clouddn.com/objecthead.png)

实例数据：是对象真正存储的有效信息。包括原生类型（primitive type）和对象引用（reference）。
对齐填充：Java对象占用空间是8字节对齐的，即所有Java对象占用bytes数必须是8的倍数。

从上面可以看出每个对象的对象头都存放着锁的信息，在堆中。
在运行期间java对象头里Mark Word里存储的数据会随着锁标志位的变化而变化。

4、 执行init方法，按照程序意愿执行
-verbose:gc -Xms20M -Xmx20M -Xmn10M -XX:+PrintGCDetails -XX:SurvivorRatio=8

## 对象实例化分析

对内存分配情况分析最常见的示例便是对象实例化:
```
Object obj = new Object();
```

这段代码的执行会涉及java栈、Java堆、方法区三个最重要的内存区域。

- obj会作为引用类型（reference）的数据保存在Java栈的本地变量表中
- Java堆中保存该引用的实例化对象
- Java堆中还必须包含能查找到此对象类型数据的地址信息（如对象类型、父类、实现的接口、方法等）
- 这些对象类型数据则保存在方法区中。

可以看出，一个对象的建立，会在堆和栈都分配空间
在堆中分配的内存实际建立对象，而栈中分配的内存只是一个指向这个堆对象的指针。


另外，由于reference类型在Java虚拟机规范里面只规定了一个指向对象的引用，并没有定义这个引用应该通过哪种方式去定位，以及访问到Java堆中的对象的具体位置，因此不同虚拟机实现的对象访问方式会有所不同，主流的访问方式有两种：使用句柄池和直接使用指针。

通过句柄池访问的方式如下：
![句柄池访问](http://7xilc8.com1.z0.glb.clouddn.com/jubing.png)

通过直接指针访问的方式如下：
![直接指针访问](http://7xilc8.com1.z0.glb.clouddn.com/zhizhen.png)



## java调优参数
-Xmx:MaxHeapSize堆内存最大的大小
-Xms:InitialHeapSize堆内存初始大小
-Xss:设置每个线程的堆栈大小
-Xmn:新生代的大小
-SuriviorRatio:新生代中Eden与Surivior的比值
比如Eden：Survivor=**3**，Survivor区有两个，即将年轻代分为**5**份，每个Survivor区占一份
-XX:PretenureSizeThreshold：大于这个数值大小的对象直接在老年代分配
-XX:MaxPermSize:永久代的最大值


# 垃圾回收

## 对象已死吗
在垃圾回收前，要判断对象是否已经死去

### 引用计数法
引用计数法很难解决相互循环引用的问题
比如obja = ojbb,objb =ojba他们互相引用，但是却再也没有其他对象引用他们

### 可达性分析算法
该方法用了从GC Roots作为起点，向下搜索，这样走过的路径叫做引用链
当一个对象到达RCRoots没有引用链的时候，证明这个对象不可用
通常GCRoots有
- 虚拟机栈中本地变量表
- 方法区中常量引用对象
- 方法区中静态类引用对象
- 本地方法栈中引用对象

### 引用
- 强引用
类似于`Object obj = new Object()`，只要强引用还在就不会被回收
- 软引用
有用但是非必须，发生系统内存溢出之前，会列入垃圾回收
- 弱引用
非必须对象，只能活到下一次回收前
- 虚拟引用
无法通过虚拟引用获得一个对象
设置幽灵引用目的是对象回收时会得到系统通知

### finalize()
在可达性分析之后，一个对象不是被被宣判死刑，而是死缓
宣告一个对象死亡要经过至少两次标记
1、 可达性分析，发现没有与GCRoots链接，标记
2、 执行过finalize()方法后，标记
被第一次标记后，还可以在finalize方法中重新建立与其他对象的联系，这就可以逃逸了
值得注意的是**finalize方法只能被执行一次**

## 垃圾收集算法

### 标记清除Mask-Sweep
![标记清除Mask-Sweep](http://7xilc8.com1.z0.glb.clouddn.com/ms.png)

不足：效率低和产生大量不连续内存碎片

### 复制算法Copying
![复制算法Copying](http://7xilc8.com1.z0.glb.clouddn.com/copy.png)


将内存分为两块，每次只使用其中的一块，这一块使用完了，就将存活的对象复制到另一块上面去
代价是将内存减半，同时如果对象存活率高，就要进行多次复制
现代的虚拟机用这种算法回收新生代
将一个内存分为较大的Eden和两个较小的Survovor区
每次使用Eden和其中一块Survior，回收时，将存活对象复制到另一个Survior区，最后清理Eden和用过的Survior
HotSpot默认的Eden和Survior比例为8:1

### 标记整理法Mask-Compact
![标记整理法Mask-Compact](http://7xilc8.com1.z0.glb.clouddn.com/mc.png)

过程与标记清除一样，但是后续不是清理而是**移动**存活对象，然后清理掉端边界以外的对象

### 分代收集算法
把java堆分为新生代Young和老年代Old

将新生代Young分为较大的Eden和两个较小的Survovor区，**新创建的对象一定都在eden中**
每次使用Eden和其中一块Survior，当Eden空间不足触发minor GC，就会将存活对象复制到另一个Survior区，最后清理Eden和用过的Survior，保证始终都有一个Surivior区是空的
HotSpot默认的Eden和Survior比例为8:1

老年代Old存放的是新生代的Survovor区满后触发minor GC仍然存活的对象。
当Eden区满后会将对象存放到Survivor区，如果Survivor区仍然存不下这些对象时，GC收集器会将这些对象放大Old区。
如果Old也满了就会出发Full GC，回收整个堆内存。

方法区中被称为是永久区Perm
永久区主要存放的是Class的类对象。
如果一个类被频繁加载，很可能会导致Perm区满。
Perm的垃圾回收是有Full GC触发的。


**新生代**每次有大量对象死去，所以用**复制法**
**老年代**存活率高，用**标记清除**或者**标记整理**

## 垃圾收集器
![垃圾收集器](http://7xilc8.com1.z0.glb.clouddn.com/%E5%9E%83%E5%9C%BE%E6%94%B6%E9%9B%86%E5%99%A8.png)
连线表示可以配合使用

### 对比

| 收集器 |  特点 | 场景 | 其他 |
| :------: |  :------: | :------: |
| Serial | 单线程 | 新生代 |经常在用在Client模式下 |
| ParNew | 并行parallel | 新生代 | 多线程版的Serial，复制算法 |
| Parellel Scavenge | 并行parallel | 新生代 | 关注吞吐量，复制算法 |
| Serial Old | 单线程 | 老年代 |多线程版的Serial，标记整理 |
| Parellel Old | 并行parallel | 老年代 |多线程版的Serial，标记整理 |
| CMS | 并发concurrent | 老年代 |目标是获得最短回收停顿，标记清除MS |
| G1 | p&C | 新生代&老年代 | 新一代收集器，将堆划分为独立区域 |

并发concurrent的关键是你有处理多个任务的能力，**不一定要同时**。
并行parallel的关键是你有**同时处理多个任务**的能力。

### GC分类

- Minor GC 新生代
当Eden区满时，触发Minor GC。因为大多数对象是朝生熄灭，所以Minor GC很频繁，而且速度很快
- Major GC / Full GC 老年代
发生在老年代的动作，出现Major GC，至少会伴随一次Minor GC，且速度比Minor GC慢10倍
（1）调用System.gc时，系统建议执行Full GC，但是不必然执行
（2）老年代空间不足
（3）方法区空间不足
（4）通过Minor GC后进入老年代的平均大小大于老年代的可用内存
（5）由Eden区、From Space区向To Space区复制时，对象大小大于To Space可用内存，则把该对象转存到老年代，且老年代的可用内存小于该对象大小

## 内存分配

### 对象优先分配在Eden新生代
### 大对象直接进入老年代
比如很长的字符串和数组
### 长期存活的对象将进入老年代
- 适用对象年龄计数器，每经过一次Minor GC年龄增加，到达一定的阈值(15)转为老年代
- 动态设定年龄的阈值，相同年龄的占一半就动态改变

## 性能分析工具
- jps
显示所有虚拟机线程
- jstat
虚拟机各方面数据
- jinfo
虚拟机配置信息
- jmap
内存快照
- jstack
线程快照

- jconsole 可视化工具
- VisualVM插件需要下载

调优经验
https://www.ibm.com/developerworks/cn/java/j-lo-jvm-optimize-experience/index.html
# 一些问题
## 内存溢出和内存泄漏
内存溢出 out of memory，是指程序在申请内存时，没有足够的内存空间供其使用，出现out of memory；比如申请了一个integer,但给它存了long才能存下的数，那就是内存溢出。
内存泄露 memory leak，是指程序在申请内存后，无法释放已申请的内存空间，一次内存泄露危害可以忽略，但内存泄露堆积后果很严重，无论多少内存,迟早会被占光。其实说白了就是该内存空间使用完毕之后未回收。
典型的内存泄漏比如我们自己用数组实现一个stack，当pop出去后
http://www.importnew.com/12961.html
https://www.ibm.com/support/knowledgecenter/en/SSEQTP_9.0.0/com.ibm.websphere.base.doc/ae/ctrb_memleakdetection.html
上面的代码实现了一个栈（先进后出（FILO））结构，乍看之下似乎没有什么明显的问题，它甚至可以通过你编写的各种单元测试。然而其中的pop方法却存在内存泄露的问题，当我们用pop方法弹出栈中的对象时，该对象不会被当作垃圾回收，即使使用栈的程序不再引用这些对象，因为栈内部维护着对这些对象的过期引用（obsolete reference）。在支持垃圾回收的语言中，内存泄露是很隐蔽的，这种内存泄露其实就是无意识的对象保持。如果一个对象引用被无意识的保留起来了，那么垃圾回收器不会处理这个对象，也不会处理该对象引用的其他对象，即使这样的对象只有少数几个，也可能会导致很多的对象被排除在垃圾回收之外，从而对性能造成重大影响，极端情况下会引发Disk Paging（物理内存与硬盘的虚拟内存交换数据），甚至造成OutOfMemoryError。
http://blog.csdn.net/jackfrued/article/details/44921941
