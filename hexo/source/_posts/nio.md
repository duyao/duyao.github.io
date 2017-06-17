title: nio
toc: true
date: 2017-04-18 16:16:57
tags: java nio
categories: note
---
# io模型
## 一些概念
### 用户空间与内核空间
现在操作系统都是采用虚拟存储器，那么对32位操作系统而言，它的寻址空间（虚拟存储空间）为4G（2的32次方）。
操作系统的核心是内核，独立于普通的应用程序，可以访问受保护的内存空间，也有访问底层硬件设备的所有权限。为了保证用户进程不能直接操作内核（kernel），保证内核的安全。
操心系统将虚拟空间划分为两部分，一部分为内核空间，一部分为用户空间。
针对Linux操作系统而言，将最高的1G字节（从虚拟地址0xC0000000到0xFFFFFFFF），供内核使用，称为内核空间，而将较低的3G字节（从虚拟地址0x00000000到0xBFFFFFFF），供各个进程使用，称为用户空间。
![用户空间和内核空间](http://7xilc8.com1.z0.glb.clouddn.com/linuxkernel.png)
### 进程切换
为了控制进程的执行，内核必须有能力挂起正在CPU上运行的进程，并恢复以前挂起的某个进程的执行。这种行为被称为进程切换。
因此可以说，任何进程都是在操作系统内核的支持下运行的，是与内核紧密相关的。

从一个进程的运行转到另一个进程上运行，这个过程中经过下面这些变化：
1. 保存处理机上下文，包括程序计数器和其他寄存器。
2. 更新PCB信息。
3. 把进程的PCB移入相应的队列，如就绪、在某事件阻塞等队列。
4. 选择另一个进程执行，并更新其PCB。
5. 更新内存管理的数据结构。
6. 恢复处理机上下文。

### 进程的阻塞
正在执行的进程，由于期待的某些事件未发生，如请求系统资源失败、等待某种操作的完成、新数据尚未到达或无新工作做等，则由系统自动执行阻塞原语(Block)，使自己由运行状态变为阻塞状态。
可见，进程的阻塞是进程自身的一种主动行为，也因此**只有处于运行态的进程（获得CPU），才可能将其转为阻塞状态**。**当进程进入阻塞状态，是不占用CPU资源的**。

### 文件描述符fd
文件描述符（File descriptor）是计算机科学中的一个术语，是一个用于表述指向文件的引用的抽象化概念。

文件描述符在形式上是一个非负整数。实际上，它是一个索引值，指向内核为每一个进程所维护的该进程打开文件的记录表。**当程序打开一个现有文件或者创建一个新文件时，内核向进程返回一个文件描述符**。在程序设计中，一些涉及底层的程序编写往往会围绕着文件描述符展开。但是文件描述符这一概念往往只适用于UNIX、Linux这样的操作系统。

### 缓存 I/O
缓存 I/O 又被称作标准 I/O，大多数文件系统的默认 I/O 操作都是缓存 I/O。
在 Linux 的缓存 I/O 机制中，操作系统会将 I/O 的数据缓存在文件系统的页缓存（ page cache ）中，也就是说，**数据会先被拷贝到操作系统内核的缓冲区中**，**然后才会从操作系统内核的缓冲区拷贝到应用程序的地址空间**。

缓存 I/O 的缺点：
数据在传输过程中需要在应用程序地址空间和内核进行多次数据拷贝操作，这些数据拷贝操作所带来的 CPU 以及内存开销是非常大的。
## io模型
刚才说了，对于一次IO访问（以read举例），数据会先被拷贝到操作系统内核的缓冲区中，然后才会从操作系统内核的缓冲区拷贝到应用程序的地址空间。所以说，当一个read操作发生时，它会经历两个阶段：
1. 等待数据准备 (Waiting for the data to be ready)
2. 将数据从内核拷贝到进程中 (Copying the data from the kernel to the process)

正式因为这两个阶段，linux系统产生了下面五种网络模式的方案。
- 阻塞I/O（blocking IO）
- 非阻塞I/O（nonblocking IO）
- I/O多路复用（ IO multiplexing）
- 异步I/O（asynchronous IO）

### 阻塞I/O
在linux中，默认情况下所有的socket都是blocking，一个典型的读操作流程大概是这样：
![阻塞I/O](http://7xilc8.com1.z0.glb.clouddn.com/blockingio.png)
当用户进程调用了recvfrom这个系统调用，kernel就开始了IO的第一个阶段：准备数据（对于网络IO来说，很多时候数据在一开始还没有到达。比如，还没有收到一个完整的UDP包。这个时候kernel就要等待足够的数据到来）。这个过程需要等待，也就是说数据被拷贝到操作系统内核的缓冲区中是需要一个过程的。而在用户进程这边，整个进程会被阻塞（当然，是进程自己选择的阻塞）。当kernel一直等到数据准备好了，它就会将数据从kernel中拷贝到用户内存，然后kernel返回结果，用户进程才解除block的状态，重新运行起来。
>blocking IO的特点就是在IO执行的两个阶段都被block了。

### 非阻塞I/O
linux下，可以通过设置socket使其变为non-blocking。当对一个non-blocking socket执行读操作时，流程是这个样子：
![非阻塞I/O](http://7xilc8.com1.z0.glb.clouddn.com/nonblockingio.png)
当用户进程发出read操作时，如果kernel中的数据还没有准备好，那么它并不会block用户进程，而是立刻返回一个error。从用户进程角度讲 ，它发起一个read操作后，并不需要等待，而是马上就得到了一个结果。用户进程判断结果是一个error时，它就知道数据还没有准备好，于是它可以再次发送read操作。一旦kernel中的数据准备好了，并且又再次收到了用户进程的system call，那么它马上就将数据拷贝到了用户内存，然后返回。
>nonblocking IO的特点是用户进程需要不断的主动询问kernel数据好了没有。

### I/O 多路复用
IO multiplexing就是我们说的select，poll，epoll，有些地方也称这种IO方式为event driven IO。
select/epoll的好处就在于单个process就可以同时处理多个网络连接的IO。它的基本原理就是select，poll，epoll这个function会不断的轮询所负责的所有socket，当某个socket有数据到达了，就通知用户进程。
![I/O 多路复用](http://7xilc8.com1.z0.glb.clouddn.com/iomultiplexing.png)
当用户进程调用了select，那么整个进程会被block，而同时，kernel会“监视”所有select负责的socket，当任何一个socket中的数据准备好了，select就会返回。这个时候用户进程再调用read操作，将数据从kernel拷贝到用户进程。

>所以，I/O 多路复用的特点是通过一种机制一个进程能同时等待多个文件描述符，而这些文件描述符（套接字描述符）其中的任意一个进入读就绪状态，select()函数就可以返回。

#### select，poll，epoll区别
1、支持一个进程所能打开的最大连接数
select有上限，poll没有上限、epoll基本没有上限
2、FD剧增后带来的IO效率问题
select和poll每次调用都要线性遍历，而epoll使用callback回调函数
3、消息传递方式
select和poll消息要从内核空间传递到用户空间，需要内核拷贝，epoll通过内核和用户空间共享一块内存空间来实现消息传递

select低效是因为每次它都需要轮询。但低效也是相对的，视情况而定，也可通过良好的设计改善
**java nio包是select模型**

https://my.oschina.net/xianggao/blog/663655
http://blog.sae.sina.com.cn/archives/5042
### 异步 I/O
Linux下的asynchronous IO其实用得很少。先看一下它的流程：
![异步 I/O](http://7xilc8.com1.z0.glb.clouddn.com/asynchronousio.png)
用户进程发起read操作之后，立刻就可以开始去做其它的事。而另一方面，从kernel的角度，当它受到一个asynchronous read之后，首先它会立刻返回，所以不会对用户进程产生任何block。然后，kernel会等待数据准备完成，然后将数据拷贝到用户内存，当这一切都完成之后，kernel会给用户进程发送一个signal，告诉它read操作完成了。

http://blog.csdn.net/ywcpig/article/details/52540853?locationNum=1&fps=1
### 几种 I/O 模型的比较
前三种模型的区别是阶段1不相同，阶段2基本相同，都是将数据从内核拷贝到调用者的缓冲区。而异步 I/O 的两个阶段都不同于三四个模型。
同步 I/O 操作引起请求进程阻塞，直到 I/O 操作完成。异步 I/O 操作不引起请求进程阻塞。
![I/O 模型比较](http://7xilc8.com1.z0.glb.clouddn.com/ComparisonIOmodels.png)
阻塞与非阻塞指的的是当不能进行读写（网卡满时的写/网卡空的时候的读）的时候，I/O 操作立即返回还是阻塞；同步异步指的是，当数据已经ready的时候，读写操作是同步读还是异步读，阶段不同而已。
## java中的io模型
阻塞I/O：服务器启动后，等待客户端连接。在客户端连接服务器后，服务器就阻塞读写取数据流。就是普通的serversocket，这种情况一个客户端连接之后，只能accept一个客户端，其他的是连接不到的

阻塞I/O+多线程：每次接收到新的连接都要新建一个线程，处理完成后销毁线程，但是这样做代价大，当有大量地短连接出现时，性能比较低。

阻塞I/O+线程池：针对上面多线程的模型中，出现的线程重复创建、销毁带来的开销，可以采用线程池来优化。每次接收到新连接后从池中取一个空闲线程进行处理，处理完成后再放回池中，重用线程避免了频率地创建和销毁线程带来的开销。
存在着问题：在大量短连接的场景中性能会有提升，因为不用每次都创建和销毁线程，而是重用连接池中的线程。但在大量长连接的场景中，因为线程被连接长期占用，不需要频繁地创建和销毁线程，因而没有什么优势。虽然这种方法可以适用于小到中度规模的客户端的并发数，如果连接数超过 100,000或更多，那么性能将很不理想。

“阻塞I/O+线程池”网络模型虽然比”阻塞I/O+多线程”网络模型在性能方面有提升，但这两种模型都存在一个共同的问题：读和写操作都是同步阻塞的,面对大并发（持续大量连接同时请求）的场景，需要消耗大量的线程来维持连接。CPU 在大量的线程之间频繁切换，性能损耗很大。一旦单机的连接超过1万，甚至达到几万的时候，服务器的性能会急剧下降。

而 NIO 的 Selector 却很好地解决了这个问题，用主线程（一个线程或者是 CPU 个数的线程）保持住所有的连接，管理和读取客户端连接的数据，将读取的数据交给后面的线程池处理，线程池处理完业务逻辑后，将结果交给主线程发送响应给客户端，少量的线程就可以处理大量连接的请求。
Java NIO 由以下几个核心部分组成：
- Channel
- Buffer
- Selector

要使用 Selector，得向 Selector 注册 Channel，然后调用它的 select()方法。这个方法会一直阻塞到某个注册的通道有事件就绪。一旦这个方法返回，线程就可以处理这些事件，事件的例子有如新连接进来，数据接收等。

异步I/O”模式
Java SE 7 版本之后，引入了异步 I/O （NIO.2） 的支持，为构建高性能的网络应用提供了一个利器。

https://waylau.com/java-io-model-evolution/#
https://github.com/waylau/essential-java/blob/master/docs/io-model.md

## 两种IO多路复用方案:Reactor and Proactor

一般情况下，I/O 复用机制需要事件分享器(event demultiplexor )。 事件分享器的作用，即将那些读写事件源分发给各读写事件的处理者，就像送快递的在楼下喊: 谁的什么东西送了， 快来拿吧。开发人员在开始的时候需要在分享器那里注册感兴趣的事件，并提供相应的处理者(event handlers)，或者是回调函数; 事件分享器在适当的时候会将请求的事件分发给这些handler或者回调函数。
涉及到事件分享器的两种模式称为：Reactor and Proactor 。
**Reactor模式是基于同步I/O的，而Proactor模式是和异步I/O相关的**。
![Reactor](http://7xilc8.com1.z0.glb.clouddn.com/reactor.png)
在Reactor模式中，事件分离者等待某个事件或者可应用或个操作的状态发生（比如文件描述符可读写，或者是socket可读写），事件分离者就把这个事件传给事先注册的事件处理函数或者回调函数，由后者来做实际的读写操作。

而在Proactor模式中，事件处理者(或者代由事件分离者发起)直接发起一个异步读写操作(相当于请求)，而实际的工作是由操作系统来完成的。发起时，需要提供的参数包括用于存放读到数据的缓存区，读的数据大小，或者用于存放外发数据的缓存区，以及这个请求完后的回调函数等信息。事件分离者得知了这个请求，它默默等待这个请求的完成，然后转发完成事件给相应的事件处理者或者回调。举例来说，在Windows上事件处理者投递了一个异步IO操作(称有overlapped的技术)，事件分离者等IOCompletion事件完成。 这种异步模式的典型实现是基于操作系统底层异步API的，所以我们可称之为“系统级别”的或者“真正意义上”的异步，因为具体的读写是由操作系统代劳的。
http://blog.jobbole.com/59676/
# nio
Java NIO（ New IO） 是从Java 1.4版本开始引入的一个新的IO API，可以替代标准的Java IO API。
NIO与原来的IO有同样的作用和目的，但是使用的方式完全不同， NIO支持面向缓冲区的、基于通道的IO操作。 NIO将以更加高效的方式进行文件的读写操作。

| IO    | NIO   |
| :------------- :| :-------------: |
|面向流(Stream Oriented)| 面向缓冲区(Buffer Oriented)|
|阻塞IO(Blocking IO)| 非阻塞IO(Non Blocking IO)|
|(无)| 选择器(Selectors)|

Java IO面向流意味着每次从流中读一个或多个字节，直至读取所有字节，它们没有被缓存在任何地方。此外，它不能前后移动流中的数据。如果需要前后移动从流中读取的数据，需要先将它缓存到一个缓冲区。
Java NIO的缓冲导向方法略有不同。数据读取到一个它稍后处理的缓冲区，需要时可在缓冲区中前后移动。这就增加了处理过程中的灵活性。但是，还需要检查是否该缓冲区中包含所有您需要处理的数据。而且，需确保当更多的数据读入缓冲区时，不要覆盖缓冲区里尚未处理的数据。

![Java NIO](http://7xilc8.com1.z0.glb.clouddn.com/nio.png)
Java NIO系统的核心在于：通道(Channel)和缓冲区(Buffer)。
通道表示打开到 IO 设备(例如：文件、套接字)的连接。若需要使用 NIO 系统，需要获取用于连接 IO 设备的通道以及用于容纳数据的缓冲区。
然后操作缓冲区，对数据进行处理。

## buffer
缓冲区（ Buffer） ：一个用于特定基本数据类型的容器。由 java.nio 包定义的，所有缓冲区都是 Buffer 抽象类的子类。
Java NIO 中的 Buffer 主要用于与 NIO 通道进行交互，数据是从通道读入缓冲区，从缓冲区写入通道中的。
Buffer 就像一个数组，可以保存多个相同类型的数据。根据数据类型不同(boolean 除外) ，有以下 Buffer 常用子类：
- ByteBuffer
- CharBuffer
- ShortBuffer
- IntBuffer
- LongBuffer
- FloatBuffer
- DoubleBuffer

上述 Buffer 类 他们都采用相似的方法进行管理数据，只是各自管理的数据类型不同而已。都是通过如下方法获取一个 Buffer对象：
static XxxBuffer allocate(int capacity) : 创建一个容量为 capacity 的 XxxBuffer 对象
### 基本属性
容量 (capacity) ： 表示 Buffer 最大数据容量，缓冲区容量不能为负，并且创建后不能更改。
限制 (limit)： 第一个不应该读取或写入的数据的索引，即位于 limit 后的数据不可读写。缓冲区的限制不能为负，并且不能大于其容量。
位置 (position)： 下一个要读取或写入的数据的索引。缓冲区的位置不能为负，并且不能大于其限制
标记 (mark)与重置 (reset)： 标记是一个索引，通过 Buffer 中的 mark() 方法指定 Buffer 中一个特定的 position，之后可以通过调用 reset() 方法恢复到这个 position.
标记、 位置、 限制、 容量遵守不变式：**0 <= mark <= position <= limit <= capacity**
![I/O 模型比较](http://7xilc8.com1.z0.glb.clouddn.com/buufer.png)

Buffer 所有子类提供了两个用于数据操作的方法： get()与 put() 方法
获取 Buffer 中的数据
- get() ：读取单个字节
- get(byte[] dst)：批量读取多个字节到 dst 中
- get(int index)：读取指定索引位置的字节(不会移动 position)
放入数据到 Buffer 中
- put(byte b)：将给定单个字节写入缓冲区的当前位置
- put(byte[] src)：将 src 中的字节写入缓冲区的当前位置
- put(int index, byte b)：将指定字节写入缓冲区的索引位置(不会移动 position)

equals()与compareTo()方法
可以使用equals()和compareTo()方法两个Buffer。

equals()当满足下列条件时，表示两个Buffer相等：
- 有相同的类型（byte、char、int等）。
- Buffer中剩余的byte、char等的个数相等。
- Buffer中所有剩余的byte、char等都相同。
如你所见，equals只是比较Buffer的一部分，不是每一个在它里面的元素都比较。实际上，它只比较Buffer中的剩余元素。

compareTo()方法比较两个Buffer的剩余元素(byte、char等)， 如果满足下列条件，则认为一个Buffer“小于”另一个Buffer：
- 第一个不相等的元素小于另一个Buffer中对应的元素 。
- 所有元素都相等，但第一个Buffer比另一个先耗尽(第一个Buffer的元素个数比另一个少)。
**剩余元素是从 position到limit之间的元素**

compact()和clear()
如果Buffer中仍有未读的数据，且后续还需要这些数据，但是此时想要先先写些数据，那么使用compact()方法。compact()方法将所有未读的数据拷贝到Buffer起始处。然后将position设到最后一个未读元素正后面。limit属性依然像clear()方法一样，设置成capacity。现在Buffer准备好写数据了，但是不会覆盖未读的数据。
如果调用的是clear()方法，position将被设回0，limit被设置成 capacity的值。换句话说，Buffer相当于 被清空了（实际上Buffer中的内容并未真正被清空，此时如果调用rewind()或者设置position=0仍然可读取旧的数据）。该方法实际上只是重设了position和limit的值，进而告诉我们可以从哪里开始往Buffer里写数据。如果Buffer中有一些未读的数据，调用clear()方法，数据将“被遗忘”，意味着不再有任何标记会告诉你哪些数据被读过，哪些还没有。
### Buffer常用方法
![Buffer常用方法](http://7xilc8.com1.z0.glb.clouddn.com/bufferfun.png)

### 直接缓冲区和非直接缓冲区
字节缓冲区要么是直接的，要么是非直接的。
![直接缓冲区和非直接缓冲区](http://7xilc8.com1.z0.glb.clouddn.com/allocateDirect.png)
非直接缓冲区：通过 allocate() 方法分配缓冲区，将缓冲区建立在 JVM 的内存中
直接缓冲区：通过 allocateDirect() 方法分配直接缓冲区，将缓冲区建立在物理内存中。可以提高效率

如果为直接字节缓冲区，则 Java 虚拟机会尽最大努力直接在此缓冲区上执行本机 I/O 操作。也就是说，在每次调用基础操作系统的一个本机 I/O 操作之前（或之后），**虚拟机都会尽量避免将缓冲区的内容复制到中间缓冲区中（或从中间缓冲区中复制内容）**。
直接字节缓冲区可以通过调用此类的 `allocateDirect()` 工厂方法来创建。此方法返回的缓冲区进行分配和取消分配所需成本通常高于非直接缓冲区。直接缓冲区的内容可以驻留在常规的垃圾回收堆之外，因此，它们对应用程序的内存需求量造成的影响可能并不明显。所以，建议将直接缓冲区主要分配给那些易受基础系统的本机 I/O 操作影响的大型、持久的缓冲区。一般情况下，最好仅在直接缓冲区能在程序性能方面带来明显好处时分配它们。

`public abstract MappedByteBuffer map(MapMode mode,long position, long size)`
直接字节缓冲区还可以通过 `FileChannel` 的 `map()` 方法 将**文件区域直接映射到物理内存中来创建**。该方法返回`MappedByteBuffer` 。 Java 平台的实现有助于通过 JNI 从本机代码创建直接字节缓冲区。如果以上这些缓冲区中的某个缓冲区实例指的是不可访问的内存区域，则试图访问该区域不会更改该缓冲区的内容，并且将会在访问期间或稍后的某个时间导致抛出不确定的异常。

字节缓冲区是直接缓冲区还是非直接缓冲区可通过调用其 `isDirect()` 方法来确定。提供此方法是为了能够在性能关键型代码中执行显式缓冲区管理。

#### HeapByteBuffer和DirectByteBuffer以及MappedByteBuffer区别
![三者的关系](http://7xilc8.com1.z0.glb.clouddn.com/mapdirectheap.png)
在`ByteBuffer.allocate(int capacity)`和`ByteBuffer.wrap(byte[] array)`中，通过实例化`HeapByteBuffer`来创建的ByteBuffer对象就是heap buffer.
在`ByteBuffer.allocateDirect(int capacity)`方法中调用`DirectByteBuffer(capacity)`这个类创建的是direct buffer

Direct Buffer则是通过JNI(native方法)在Java的虚拟机外的内存中分配了一块缓冲区(所以即使在运行时通过-Xmx指定了Java虚拟机的最大堆内存，还是可以实例化超出该大小的Direct ByteBuffer),该块并不直接由Java虚拟机负责垃圾回收收集，但是在direct buffer包装类`DirectByteBuffer`被回收时，会通过Java Reference机制来释放该内存块。(但Direct Buffer的JAVA对象`DirectByteBuffer`是归GC管理的，只要GC回收了它的JAVA对象，操作系统才会释放Direct Buffer所申请的空间)。
DirectByteBuffer 自身是一个Java对象，在Java堆中；而这个对象中有个long类型字段address，记录着一块调用 malloc() 申请到的native memory。
```
			 Java        |      native
                   |
 DirectByteBuffer  |     malloc'd
 [    address   ] -+-> [   data    ]
                   |
```
**DirectByteBuffer 自身是（Java）堆内的，它背后真正承载数据的buffer是在（Java）堆外——native memory中的**。这是 malloc() 分配出来的内存，是用户态的。
https://www.zhihu.com/question/57374068/answer/152691891


**heap buffer这种缓冲区是分配在堆上面的，直接由Java虚拟机负责垃圾回收**，可以直接想象成一个字节数组的包装类。
FileChannel 的read(ByteBuffer dst)函数,write(ByteBuffer src)函数中，**如果传入的参数是HeapBuffer类型,则会临时申请一块DirectBuffer,进行数据拷贝，而不是直接进行数据传输.**

劣势：创建和释放Direct Buffer的代价比Heap Buffer得要高；
优势：当我们把一个Direct Buffer写入Channel的时候，就好比是“内核缓冲区”的内容直接写入了Channel，这样显然快了，减少了数据拷贝（因为我们平时的read/write都是需要在I/O设备与应用程序空间之间的“内核缓冲区”中转一下的）。而当我们把一个Heap Buffer写入Channel的时候，实际上底层实现会先构建一个临时的Direct Buffer，然后把Heap Buffer的内容复制到这个临时的Direct Buffer上，再把这个Direct Buffer写出去。当然，如果我们多次调用write方法，把一个Heap Buffer写入Channel，底层实现可以重复使用临时的Direct Buffer，这样不至于因为频繁地创建和销毁Direct Buffer影响性能。

结论：Direct Buffer创建和销毁的代价很高，所以要用在尽可能重用的地方。 比如周期长传输文件大采用direct buffer，不然一般情况下就直接用heap buffer 就好。
http://eyesmore.iteye.com/blog/1133335


>The difference between the buffer types is that MappedByteBuffers are allocated in virtual-memory space in the operating system. R/W done with MappedByteBuffers is managed at the OS level by the VM paging logic. Direct ByteBuffers are just a solid slab of free memory (e.g. malloc) in RAM that you can utilize from within Java and treated by the OS as a standard memory allocation.
A MappedByteBuffer represents a section of memory allocated using mmap call, which is used to perform memory mapped I/O. Therefore MappedByteBuffers won't register their use of memory in the same way a Direct ByteBuffer will.

http://www.developersite.org/903-129308-file
http://stackoverflow.com/questions/1229037/difference-between-bytebuffer-allocatedirect-and-mappedbytebuffer-load
DirectByteBuffers是操作系统中真实的区域，通过malloc在用户空间申请的内存区域，并不是在jvm中
MappedByteBuffers申请了操作系统中的虚拟内存，当这块buffer进行读写操作的时候，需要操作系统进行页的置换，并没有真正的申请内存，只是调用了mmap()方法

**内存映射文件和标准IO操作最大的不同之处就在于它虽然最终也是要从磁盘读取数据，但是它并不需要将数据读取到OS内核缓冲区，而是直接将进程的用户私有地址空间中的一部分区域与文件对象建立起映射关系**，就好像直接从内存中读、写文件一样，速度当然快了
Linux只能怪有块区域叫做“Memory mapped region for shared libraries” ，这段区域就是在内存映射文件的时候将某一段的虚拟地址和文件对象的某一部分建立起映射关系，此时并没有拷贝数据到内存中去，而是当进程代码第一次引用这段代码内的虚拟地址时，触发了缺页异常，这时候OS根据映射关系直接将文件的相关部分数据拷贝到进程的用户私有空间中去，当有操作第N页数据的时候重复这样的OS页面调度程序操作。注意啦，原来内存映射文件的效率比标准IO高的重要原因就是因为少了把数据拷贝到OS内核缓冲区这一步

http://blog.csdn.net/fcbayernmunchen/article/details/8635427

## channel
![数据的传输过程](http://7xilc8.com1.z0.glb.clouddn.com/bufferchannel.png)
通道（ Channel）：由 java.nio.channels 包定义的。 Channel 表示 IO 源与目标打开的连接。Channel 类似于传统的“流”。只不过 Channel本身不能直接访问数据， Channel 只能与Buffer 进行交互。

Java 为 Channel 接口提供的最主要实现类如下：
- FileChannel：用于读取、写入、映射和操作文件的通道。
- DatagramChannel：通过 UDP 读写网络中的数据通道。
- SocketChannel：通过 TCP 读写网络中的数据。
- ServerSocketChannel：可以监听新进来的 TCP 连接，对每一个新进来的连接都会创建一个 SocketChannel。

获取通道的一种方式是对支持通道的对象调用getChannel() 方法。支持通道的类如下：
- FileInputStream
- FileOutputStream
- RandomAccessFile
- DatagramSocket
- Socket
- ServerSocket

获取通道的其他方式是使用 Files 类的静态方法 newByteChannel() 获取字节通道。
`FileChannel channel = new RandomAccessFile("1.txt", "rw").getChannel()`
或者通过通道的静态方法 open() 打开并返回指定通道。
`public static FileChannel open(Path path, OpenOption... options) throws IOException`
![FileChannel常见用法](http://7xilc8.com1.z0.glb.clouddn.com/filechannel.png)
### 通道的数据传输
通道将数据传输给ByteBuffer对象或者从ByteBuffer对象获取数据进行传输
通道可以是单向(unidirectional)或者双向的(bidirectional)。一个channel类可能实现定义read()方法的ReadableByteChannel接口，而另一个channel类也许实现WritableByteChannel接口以提供write()方法。实现这两种接口其中之一的类都是单向的，只能在一个方向上传输数据。如果一个类同时实现这两个接口，那么它是双向的，可以双向传输数据。

标准io
```java
public void test1(){

	FileInputStream fis = null;
	FileOutputStream fos = null;
	//①获取通道
	FileChannel inChannel = null;
	FileChannel outChannel = null;
	try {
		fis = new FileInputStream("d:/1.mkv");
		fos = new FileOutputStream("d:/2.mkv");

		inChannel = fis.getChannel();
		outChannel = fos.getChannel();

		//②分配指定大小的缓冲区
		ByteBuffer buf = ByteBuffer.allocate(1024);

		//③将通道中的数据存入缓冲区中
		while(inChannel.read(buf) != -1){
			buf.flip(); //切换读取数据的模式
			//④将缓冲区中的数据写入通道中
			outChannel.write(buf);
			buf.clear(); //清空缓冲区
		}
	} catch (IOException e) {
		e.printStackTrace();
	} finally {
		if(outChannel != null){
			try {
				outChannel.close();
			} catch (IOException e) {
				e.printStackTrace();
			}
		}

		if(inChannel != null){
			try {
				inChannel.close();
			} catch (IOException e) {
				e.printStackTrace();
			}
		}

		if(fos != null){
			try {
				fos.close();
			} catch (IOException e) {
				e.printStackTrace();
			}
		}

		if(fis != null){
			try {
				fis.close();
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	}

}
```
直接io
```java
public void test2() throws IOException{

	FileChannel inChannel = FileChannel.open(Paths.get("d:/1.mkv"), StandardOpenOption.READ);
	FileChannel outChannel = FileChannel.open(Paths.get("d:/2.mkv"), StandardOpenOption.WRITE, StandardOpenOption.READ, StandardOpenOption.CREATE);

	//内存映射文件
	MappedByteBuffer inMappedBuf = inChannel.map(MapMode.READ_ONLY, 0, inChannel.size());
	MappedByteBuffer outMappedBuf = outChannel.map(MapMode.READ_WRITE, 0, inChannel.size());

	//直接对缓冲区进行数据的读写操作
	byte[] dst = new byte[inMappedBuf.limit()];
	inMappedBuf.get(dst);
	outMappedBuf.put(dst);

	inChannel.close();
	outChannel.close();
}
```
### 分散(Scatter)和聚集(Gather)
分散读取（ Scattering Reads）是指从 Channel 中读取的数据“分散” 到多个 Buffer 中。
注意：按照缓冲区的顺序，从 Channel 中读取的数据依次将 Buffer 填满。
`public final long read(ByteBuffer[] dsts) throws IOException`
聚集写入（ Gathering Writes）是指将多个 Buffer 中的数据“聚集”到 Channel。
注意：按照缓冲区的顺序，写入 position 和 limit 之间的数据到 Channel 。
`public final long write(ByteBuffer[] srcs) throws IOException`
### 通道之间的传输

`public abstract long transferFrom(ReadableByteChannel src, long position, long count)`
`public abstract long transferTo(long position, long count, WritableByteChannel target)`
```java
public void test3() throws IOException{
	FileChannel inChannel = FileChannel.open(Paths.get("d:/1.mkv"), StandardOpenOption.READ);
	FileChannel outChannel = FileChannel.open(Paths.get("d:/2.mkv"), StandardOpenOption.WRITE, StandardOpenOption.READ, StandardOpenOption.CREATE);

//		inChannel.transferTo(0, inChannel.size(), outChannel);
	outChannel.transferFrom(inChannel, 0, inChannel.size());

	inChannel.close();
	outChannel.close();
}
```

### 网络channel

ServerSocketChannel： Java NIO中的 ServerSocketChannel 是一个可以监听新进来的TCP连接的通道，就像标准IO中的ServerSocket一样。

SocketChannel：Java NIO中的SocketChannel是一个连接到TCP网络套接字的通道。
操作步骤：① 打开 SocketChannel ② 读写数据 ③ 关闭 SocketChanne

DatagramChannel：Java NIO中的DatagramChannel是一个能收发UDP包的通道。
操作步骤： ① 打开 DatagramChannel ② 接收/发送数据

## Selector

选择器（ Selector） 是 `SelectableChannle` 对象的多路复用器， Selector 可以同时监控多个 SelectableChannel 的 IO 状况，也就是说，利用 Selector可使一个单独的线程管理多个 Channel。 Selector 是非阻塞 IO 的核心。
![SelectableChannle](http://7xilc8.com1.z0.glb.clouddn.com/selectablechannel.png)
由此可见selector只能监控网络channel

Selector 的常用方法
![Selector](http://7xilc8.com1.z0.glb.clouddn.com/selector.png)

### SelectionKey
SelectionKey： 表示 SelectableChannel 和 Selector 之间的注册关系。
每次向选择器注册通道时就会选择一个事件(选择键)。 选择键包含两个表示为整数值的操作集。操作集的每一位都表示该键的通道所支持的一类可选择操作。
可以监听的事件类型（ 可使用 SelectionKey 的四个常量表示）：
读 : SelectionKey.OP_READ （ 1）
写 : SelectionKey.OP_WRITE （ 4）
连接 : SelectionKey.OP_CONNECT （ 8）
接收 : SelectionKey.OP_ACCEPT （ 16）
![SelectionKey](http://7xilc8.com1.z0.glb.clouddn.com/selelctablekey.png)

### 选择器（ Selector）的应用
1.创建 Selector ：通过调用 `Selector selector = Selector.open()` 方法创建一个 Selector。
2.向选择器注册通道： `SelectableChannel.register(Selector sel, int ops)`
当调用 `register(Selector sel, int ops)` 将通道注册选择器时，选择器对通道的监听事件，需要通过第二个参数 ops 指定。

若注册时不止监听一个事件，则可以使用“位或”操作符连接。
`SelectableChannel.register(selector, SelectionKey.OP_ACCEPT|SelectionKey.OP_READ)`
### NIO中的server
```java
public void server() throws IOException{
	//1. 获取通道
	ServerSocketChannel ssChannel = ServerSocketChannel.open();

	//2. 切换非阻塞模式
	ssChannel.configureBlocking(false);

	//3. 绑定连接
	ssChannel.bind(new InetSocketAddress(9898));

	//4. 获取选择器
	Selector selector = Selector.open();

	//5. 将通道注册到选择器上, 并且指定“监听接收事件”
	ssChannel.register(selector, SelectionKey.OP_ACCEPT);

	//6. 轮询式的获取选择器上已经“准备就绪”的事件
	while(selector.select() > 0){

		//7. 获取当前选择器中所有注册的“选择键(已就绪的监听事件)”
		Iterator<SelectionKey> it = selector.selectedKeys().iterator();

		while(it.hasNext()){
			//8. 获取准备“就绪”的是事件
			SelectionKey sk = it.next();

			//9. 判断具体是什么事件准备就绪
			if(sk.isAcceptable()){
				//10. 若“接收就绪”，获取客户端连接
				SocketChannel sChannel = ssChannel.accept();

				//11. 切换非阻塞模式
				sChannel.configureBlocking(false);

				//12. 将该通道注册到选择器上
				sChannel.register(selector, SelectionKey.OP_READ);
			}else if(sk.isReadable()){
				//13. 获取当前选择器上“读就绪”状态的通道
				SocketChannel sChannel = (SocketChannel) sk.channel();

				//14. 读取数据
				ByteBuffer buf = ByteBuffer.allocate(1024);

				int len = 0;
				while((len = sChannel.read(buf)) > 0 ){
					buf.flip();
					System.out.println(new String(buf.array(), 0, len));
					buf.clear();
				}
			}

			//15. 取消选择键 SelectionKey
			it.remove();
		}
	}
}
```
# 管道 (Pipe)
Java NIO 管道是2个线程之间的单向数据连接。
Pipe有一个source通道和一个sink通道。**数据会被写到sink通道，从source通道读取**。
![SelectionKey](http://7xilc8.com1.z0.glb.clouddn.com/pipe.png)
```java
public void test1() throws IOException{
	//1. 获取管道
	Pipe pipe = Pipe.open();

	//2. 将缓冲区中的数据写入管道
	ByteBuffer buf = ByteBuffer.allocate(1024);

	Pipe.SinkChannel sinkChannel = pipe.sink();
	buf.put("通过单向管道发送数据".getBytes());
	buf.flip();
	sinkChannel.write(buf);

	//3. 读取缓冲区中的数据
	Pipe.SourceChannel sourceChannel = pipe.source();
	buf.flip();
	int len = sourceChannel.read(buf);
	System.out.println(new String(buf.array(), 0, len));

	sourceChannel.close();
	sinkChannel.close();
}
```
# Path、 Paths、 Files、CharSet

http://www.365mini.com/page/tag/java-nio
http://ifeve.com/?x=0&y=0&s=nio
