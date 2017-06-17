title: juc之同步工具和并发容器
toc: true
date: 2017-03-24 13:40:21
tags: jvm
categories: note
---
# 同步工具
http://watchmen.cn/portal.php?mod=view&aid=513

##  ReentrantLock

### 可中断
public void lockInterruptibly() throws InterruptedException
调用`lockInterruptibly()`声明可中断
### 可限时
public boolean tryLock()
public boolean tryLock(long timeout, TimeUnit unit) throws InterruptedException
超时不能获得锁，就返回false，不会永久等待构成死锁
### 公平锁
先来先得
public ReentrantLock(boolean fair)
public static ReentrantLock fairLock = new ReentrantLock(true);
### 可绑定条件
ReentrantLock可以和多个condition一起使用
### 与synchronized区别
主要区别就是ReentrantLock的四个特性：可中断、可限时、公平锁、可绑定多个条件
synchronized比较简单，通常与一个对象相关联，悲观锁，
ReentrantLock更高级，但是比较复杂，可以与多个对象相关联，乐观锁
他们都是可重入的，synchronized子程序调用父类，不会产生死锁的。
```java
public class Widget {
    public synchronized void doSomething() {
        ...
    }
}

public class LoggingWidget extends Widget {
    public synchronized void doSomething() {
        System.out.println(toString() + ": calling doSomething");
        super.doSomething();//若内置锁是不可重入的，则发生死锁
    }
}
```
http://topmanopensource.iteye.com/blog/1736739
http://www.importnew.com/20472.html


### 实现原理
应用层面的锁，基本上使用java实现的，很少有很底层的东西
- CAS原理
- 等待队列
- LockSupport.park();

基于AQS的锁(比如ReentrantLock)原理大体是这样:
有一个state变量，初始值为0，假设当前线程为A,每当A获取一次锁，status++. 释放一次，status--.锁会记录当前持有的线程。
当A线程拥有锁的时候，status>0. B线程尝试获取锁的时候会对这个status有一个CAS(0,1)的操作，尝试几次失败后就挂起线程，进入一个等待队列。
如果A线程恰好释放，--status==0, A线程会去唤醒等待队列中第一个线程，即刚刚进入等待队列的B线程，B线程被唤醒之后回去检查这个status的值，尝试CAS(0,1),而如果这时恰好C线程也尝试去争抢这把锁

非公平锁实现：
C直接尝试对这个status CAS(0,1)操作，并成功改变了status的值，B线程获取锁失败，再次挂起，这就是非公平锁，B在C之前尝试获取锁，而最终是C抢到了锁。
公平锁：
C发现有线程在等待队列，直接将自己进入等待队列并挂起,B获取锁

http://www.cnblogs.com/xrq730/p/4979021.html
http://www.importnew.com/19472.html
http://blog.csdn.net/ns_code/article/details/17487337
https://www.zhihu.com/question/36964449/answer/69790971?utm_source=com.youdao.note&utm_medium=social
http://www.cnblogs.com/maxmys/p/5181775.html
## ReadWriteLock
### StampedLock

##  Condition
条件对象：进入临界区时发现必须满足一定的条件才能执行，那么就可以使用一个条件对象管理那些已经获得锁但是不能工作的线程
类似于 Object.wait()和Object.notify()
与ReentrantLock结合使用，一个ReentrantLock可以有多个Condition，习惯上给条件对象命名为可以反应它所表达的条件的名字

```
public void transfer(int from, int amount){
  ReentrantLock bank = new ReentrantLock();
  Condition sufficient = bank.newCondition();

  bank.lock();//如果使用锁就不能使用带资源的try语句
  try {
    while(!account[from] < account){
      sufficient.await();//await调用必须方法while(!ok to proceed)中，阻塞线程
      //transfer funds

      sufficient.signalAll();//解除等待线程的阻塞
  }
  }finally {
    bank.lock();//解锁一定要放在finally中
  }
}

```

### 主要方法
- void await() throws InterruptedException;
await()方法会使当前线程等待，同时释放当前锁，当其他线程中使用signal()时或者signalAll()方法时，线程会重新获得锁并继续执行。或者当线程被中断时，也能跳出等待。这和Object.wait()方法很相似。

- void awaitUninterruptibly();
awaitUninterruptibly()方法与await()方法基本相同，但是它并不会再等待过程中响应中断。

- void signal();
singal()方法用于唤醒一个在等待中的线程。

- void signalAll();
相对的singalAll()方法会唤醒所有在等待中的线程。这和Obejct.notify()方法很类似。

await、signal、signalAll必须抛异常

## Semaphore信号量
共享锁，运行多个线程同时临界

### 主要接口
public void acquire()
public void acquireUninterruptibly()
public boolean tryAcquire()
public boolean tryAcquire(long timeout, TimeUnit unit)
public void release()


```java
public class SemaphoreDemo implements Runnable {
    //只有两个信号量
    final Semaphore semaphore = new Semaphore(2);
    static SemaphoreDemo demo = new SemaphoreDemo();

    public static void main(String[] args) {
        //有10个线程去抢夺
        ExecutorService executor = Executors.newFixedThreadPool(10);
        for (int i = 0; i < 10; i++) {
            executor.submit(demo);
        }

    }

    @Override
    public void run() {
        try {
            //获得
            semaphore.acquire();
            Thread.sleep(1000);
            System.out.println("Thread " + Thread.currentThread().getId() + " done!");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            //释放
            semaphore.release();
        }

    }
}

```

## CountDownLatch
倒数计时器
一种典型的场景就是火箭发射。在火箭发射前，为了保证万无一失，往往还要进行各项设备、仪器的检查。
只有等所有检查完毕后，引擎才能点火。这种场景就非常适合使用CountDownLatch。它可以使得点火线程，等待所有检查线程全部完工后，再执行

### 使用
static final CountDownLatch end = new CountDownLatch(10);
end.countDown();
end.await();

```java
public class CountDownLatchDemo implements Runnable {
    //5个任务需要检查
    static final CountDownLatch end = new CountDownLatch(5);
    static final CountDownLatchDemo demo = new CountDownLatchDemo();

    public static void main(String[] args) throws InterruptedException {
        //执行5个线程完成检查，如果没有完成，所有线程就会阻塞，等待完成
        // 所以线程的个数必须要与倒计时个数相同
        ExecutorService executorService = Executors.newFixedThreadPool(5);
        for (int i = 0; i < 5; i++) {
            executorService.submit(demo);
        }
        //等待通知
        end.await();
        System.out.println("Fired");
        executorService.shutdown();



    }

    @Override
    public void run() {
        Random r = new Random();
        try {
            Thread.sleep(r.nextInt(5) * 1000);
            end.countDown();
            System.out.println("chenk over, remaining "+end.getCount() );
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

    }
}
```

##  CyclicBarrier
循环栅栏
Cyclic意为循环，也就是说这个计数器可以反复使用。比如，假设我们将计数器设置为10。那么凑齐第一批10个线程后，计数器就会归零，然后接着凑齐下一批10个线程
### 主要接口
- public CyclicBarrier(int parties, Runnable barrierAction)
barrierAction就是当计数器一次计数完成后，系统会执行的动作，必须是实现Runnable接口

-  public int await() throws InterruptedException, BrokenBarrierException
抛出InterruptedException中断异常的目的是避免线程中断，而一直阻塞，产生永久性的异常
抛出BrokenBarrierException的原因是一批线程中只有凑够了个数才会执行。
事实上，有可能出现其中一个线程出问题，那么导致其他线程阻塞无法执行，这时候其他线程就会抛出BrokenBarrierException，表示自己永远不能执行了

```java
public class CyclicBarrierDemo {
    public static class Soldier implements Runnable {
        private String soldier;
        private final CyclicBarrier cyclic;

        public Soldier(CyclicBarrier cyclic, String soldier) {
            this.soldier = soldier;
            this.cyclic = cyclic;
        }


        @Override
        public void run() {
            try {
                //复用栅栏
                //等待所有士兵到齐
                cyclic.await();
                doWork();
                //等待所有士兵完成工作
                cyclic.await();
            } catch (InterruptedException e) {//在等待过程中,线程被中断
                e.printStackTrace();
            } catch (BrokenBarrierException e) {//表示当前CyclicBarrier已经损坏.系统无法等到所有线程到齐了.
                e.printStackTrace();
            }
        }

        void doWork() {
            try {
                Thread.sleep(Math.abs(new Random().nextInt() % 10000));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(soldier + ":任务完成");
        }

    }

    public static class BarrierRun implements Runnable {
        boolean flag;
        int N;

        public BarrierRun(boolean flag, int N) {
            this.flag = flag;
            this.N = N;
        }


        @Override
        public void run() {
            if (flag) {
                System.out.println("司令:[士兵" + N + "个,任务完成!]");
            } else {
                System.out.println("司令:[士兵" + N + "个,集合完毕!]");
                flag = true;
            }
        }
    }

    public static void main(String[] args) {
        final int N = 10;
        Thread[] allSoldier = new Thread[N];
        boolean flag = false;
        //第二个参数就是当计数器一次计数完成后，系统会执行的动作
        CyclicBarrier cyclic = new CyclicBarrier(N, new BarrierRun(flag, N));
        //设置屏障点,主要为了执行这个方法
        System.out.println("集合队伍! ");
        for (int i = 0; i < N; i++) {
            System.out.println("士兵" + i + "报道! ");
            allSoldier[i] = new Thread(new Soldier(cyclic, "士兵" + i));
            allSoldier[i].start();

//            if(i == 5){
//               allSoldier[i].interrupt();
//            }
        }
    }
}


```

### 区别
CyclicBarrier是支持复用的，协调的是多个线程之间的顺序，即**线程之间要互相等待**，比如abc分别执行任务，然后等三个都完成之后才能继续执行新的任务
CountDownLatch是**一个线程等待其他多个线程的完成**

## LockSupport
提供线程阻塞原语
与suspend()比较，不容易引起线程冻结

同时能够响应中断，但不抛出异常。
中断响应的结果是，park()函数的返回，可以从Thread.interrupted()得到中断标志

使用的比较底层的操作，类似于被广泛的应用在其他类的是实现中
### 主要接口
LockSupport.park();
LockSupport.unpark(Thread thread);

## 线程之间的通信
http://wingjay.com/2017/04/09/Java%E9%87%8C%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0%E7%BA%BF%E7%A8%8B%E9%97%B4%E9%80%9A%E4%BF%A1%EF%BC%9F/


# 并发容器
对于容器map、set、list等，如果要实现同步，可以使用`Collections.synchronizedMap(map)`,`Collections.synchronizedList(list)`d等方法但是这仅仅适合并发量小的情况。
因为在其内部，使用`synchronized`控制`final Object mutex`变量的获取。
每个方法在执行前，都会先获取mutex然后再执行，也就是这个map每次只能被一个线程操作，这样每个方法的执行过程相对就串行化，比如两个get完全不用获得锁
```java
private static class SynchronizedMap<K,V> implements Map<K,V>, Serializable{

    private final Map<K,V> m;     // Backing Map
    final Object      mutex;        // Object on which to synchronize
    public V get(Object key) {
       synchronized (mutex) {return m.get(key);}
    }

    public V put(K key, V value) {
       synchronized (mutex) {return m.put(key, value);}
    }
    public V remove(Object key) {
       synchronized (mutex) {return m.remove(key);}
    }
}
```
## ConcurrentHashMap
高性能HashMap
![ConcurrentHashMap](http://7xilc8.com1.z0.glb.clouddn.com/segment.jpg)
ConcurrentHashMap将hash表分为16个桶（默认值），诸如get,put,remove等常用操作只锁当前需要用到的桶。
试想，原来只能一个线程进入，现在却能同时16个写线程进入（写线程才需要锁定，而读线程几乎不受限制，之后会提到），并发性的提升是显而易见的。
更令人惊讶的是ConcurrentHashMap的读取并发，因为在**读取的大多数时候都没有用到锁定，所以读取操作几乎是完全的并发操作**，而**写操作锁定的粒度又非常细**，比起之前又更加快速（这一点在桶更多时表现得更明显些）。
**ConcurrentHashMap只有在求size等操作时才需要锁定整个表**
### java1.7实现方法
> 锁分离 (Lock Stripping)

ConcurrentHashMap允许多个修改操作并发进行，其关键在于使用了锁分离技术。它使用了多个锁来控制对hash表的不同部分进行的修改。ConcurrentHashMap内部使用段(Segment)来表示这些不同的部分，每个段其实就是一个小的hash table，它们有自己的锁。只要多个修改操作发生在不同的段上，它们就可以并发进行。
> 不变(Immutable)和易变(Volatile)

ConcurrentHashMap完全允许多个读操作并发进行，读操作并不需要加锁。如果使用传统的技术，如HashMap中的实现，如果允许可以在hash链的中间添加或删除元素，读操作不加锁将得到不一致的数据。ConcurrentHashMap实现技术是保证HashEntry几乎是不可变的。HashEntry代表每个hash链中的一个节点，其结构如下所示：
```java
static final class HashEntry<K,V> {  
    final K key;  
    final int hash;  
    volatile V value;  
    final HashEntry<K,V> next;  
}
```
可以看到除了value不是final的，其它值都是final的，这意味着不能从hash链的中间或尾部添加或删除节点，因为这需要修改next引用值，所有的节点的修改只能从头部开始。对于put操作，可以一律添加到Hash链的头部。但是对于remove操作，可能需要从中间删除一个节点，这就需要将要删除节点的前面所有节点整个复制一遍，最后一个节点指向要删除结点的下一个结点。这在讲解删除操作时还会详述。为了确保读操作能够看到最新的值，将value设置成volatile，这避免了加锁。

ConcurrentHashMap是由Segment数组结构和HashEntry数组结构组成。
Segment是一种可重入锁ReentrantLock，在ConcurrentHashMap里扮演锁的角色，HashEntry则用于存储键值对数据。
```java
static final class Segment<K,V> extends ReentrantLock implements Serializable {  
private static final long serialVersionUID = 2249069246763182397L;  
     /**
      * The number of elements in this segment's region.
      */  
     transient volatile int count;  

     /**
      * Number of updates that alter the size of the table. This is
      * used during bulk-read methods to make sure they see a
      * consistent snapshot: If modCounts change during a traversal
      * of segments computing size or checking containsValue, then
      * we might have an inconsistent view of state so (usually)
      * must retry.
      */  
     transient int modCount;  

     /**
      * The table is rehashed when its size exceeds this threshold.
      * (The value of this field is always <tt>(int)(capacity *
      * loadFactor)</tt>.)
      */  
     transient int threshold;  

     /**
      * The per-segment table.
      */  
     transient volatile HashEntry<K,V>[] table;  

     /**
      * The load factor for the hash table.  Even though this value
      * is same for all segments, it is replicated to avoid needing
      * links to outer object.
      * @serial
      */  
     final float loadFactor;  
}
```
一个ConcurrentHashMap里包含一个Segment数组，Segment的结构和HashMap类似，是一种数组和链表结构， 一个Segment里包含一个HashEntry数组，每个HashEntry是一个链表结构的元素， 每个Segment守护者一个HashEntry数组里的元素,当对HashEntry数组的数据进行修改时，必须首先获得它对应的Segment锁。
#### get操作
```java
V get(Object key, int hash) {  
    if (count != 0) { // read-volatile  
        HashEntry<K,V> e = getFirst(hash);  
        while (e != null) {  
            if (e.hash == hash && key.equals(e.key)) {  
                V v = e.value;  
                if (v != null)  
                    return v;  
                return readValueUnderLock(e); // recheck  
            }  
            e = e.next;  
        }  
    }  
    return null;  
}  
```
get操作不需要锁。
第一步是访问count变量，这是一个volatile变量，由于所有的修改操作在进行结构修改时都会在最后一步写count变量，通过这种机制保证get操作能够得到几乎最新的结构更新。对于非结构更新，也就是结点值的改变，由于HashEntry的value变量是volatile的，也能保证读取到最新的值。
接下来就是对hash链进行遍历找到要获取的结点，如果没有找到，直接访回null。

对hash链进行遍历不需要加锁的原因在于链指针next是final的。
但是头指针却不是final的，这是通过getFirst(hash)方法返回，也就是存在table数组中的值。这使得getFirst(hash)可能返回过时的头结点，例如，当执行get方法时，刚执行完getFirst(hash)之后，另一个线程执行了删除操作并更新头结点，这就导致get方法中返回的头结点不是最新的。这是可以允许，通过对count变量的协调机制，get能读取到几乎最新的数据，虽然可能不是最新的。要得到最新的数据，只有采用完全的同步。

最后，如果找到了所求的结点，判断它的值如果非空就直接返回，否则在有锁的状态下再读一次。
#### put操作
```java
V put(K key, int hash, V value, boolean onlyIfAbsent) {  
    lock();  
    try {  
        int c = count;  
        if (c++ > threshold) // ensure capacity  
            rehash();  
        HashEntry<K,V>[] tab = table;  
        int index = hash & (tab.length - 1);  
        HashEntry<K,V> first = tab[index];  
        HashEntry<K,V> e = first;  
        while (e != null && (e.hash != hash || !key.equals(e.key)))  
            e = e.next;  

        V oldValue;  
        if (e != null) {  
            oldValue = e.value;  
            if (!onlyIfAbsent)  
                e.value = value;  
        }  
        else {  
            oldValue = null;  
            ++modCount;  
            tab[index] = new HashEntry<K,V>(key, hash, first, value);  
            count = c; // write-volatile  
        }  
        return oldValue;  
    } finally {  
        unlock();  
    }  
}  
```
该方法也是在持有段锁的情况下执行的，首先判断是否需要rehash，需要就先rehash。
接着是找是否存在同样一个key的结点，如果存在就直接替换这个结点的值。否则创建一个新的结点并添加到hash链的头部，这时一定要修改modCount和count的值，同样修改count的值一定要放在最后一步。put方法调用了rehash方法，reash方法实现得也很精巧，主要利用了table的大小为2^n，这里就不介绍了。

#### remove操作

整个操作是在持有段锁的情况下执行的，空白行之前的行主要是定位到要删除的节点e。
接下来，如果不存在这个节点就直接返回null，否则就要将**e前面的结点复制一遍**，尾结点指向e的下一个结点。**e后面的结点不需要复制**，它们可以重用。

```java
V remove(Object key, int hash, Object value) {  
    lock();  
    try {  
        int c = count - 1;  
        HashEntry<K,V>[] tab = table;  
        int index = hash & (tab.length - 1);  
        HashEntry<K,V> first = tab[index];  
        HashEntry<K,V> e = first;  
        while (e != null && (e.hash != hash || !key.equals(e.key)))  
            e = e.next;  

        V oldValue = null;  
        if (e != null) {  
            V v = e.value;  
            if (value == null || value.equals(v)) {  
                oldValue = v;  
                // All entries following removed node can stay  
                // in list, but all preceding ones need to be  
                // cloned.  
                ++modCount;  
                HashEntry<K,V> newFirst = e.next;  
                //复制
                for (HashEntry<K,V> p = first; p != e; p = p.next)  
                    newFirst = new HashEntry<K,V>(p.key, p.hash,  
                                                  newFirst, p.value);  
                tab[index] = newFirst;  
                count = c; // write-volatile  
            }  
        }  
        return oldValue;  
    } finally {  
        unlock();  
    }  
}
```

![remove方法](http://7xilc8.com1.z0.glb.clouddn.com/concurrenthashmapremove.png)

参考：

http://www.iteye.com/topic/344876
http://www.infoq.com/cn/articles/ConcurrentHashMap


### java1.8实现方法

改进一：取消segments字段，直接采用transient volatile HashEntry<K,V>[] table保存数据，采用table数组元素作为锁，从而实现了对每一行数据进行加锁，进一步减少并发冲突的概率。

改进二：将原先table数组＋单向链表的数据结构，变更为table数组＋单向链表＋红黑树的结构。对于hash表来说，最核心的能力在于将key hash之后能均匀的分布在数组中。如果hash之后散列的很均匀，那么table数组中的每个队列长度主要为0或者1。但实际情况并非总是如此理想，虽然ConcurrentHashMap类默认的加载因子为0.75，但是在数据量过大或者运气不佳的情况下，还是会存在一些队列长度过长的情况，如果还是采用单向列表方式，那么查询某个节点的时间复杂度为O(n)；因此，对于个数超过8(默认值)的列表，jdk1.8中采用了红黑树的结构，那么查询的时间复杂度可以降低到O(logN)，可以改进性能。

http://blog.csdn.net/u010412719/article/details/52145145
http://www.cnblogs.com/everSeeker/p/5601861.html

## BlockingQueue
阻塞队列，不是高性能，接口
可以用于生产者消费者模式

```java
class Producer implements Runnable {
  private final BlockingQueue queue;
  Producer(BlockingQueue q) { queue = q; }
  public void run() {
    try {
      while (true) { queue.put(produce()); }
    } catch (InterruptedException ex) { ... handle ...}
  }
  Object produce() { ... }
}

class Consumer implements Runnable {
  private final BlockingQueue queue;
  Consumer(BlockingQueue q) { queue = q; }
  public void run() {
    try {
      while (true) { consume(queue.take()); }
    } catch (InterruptedException ex) { ... handle ...}
  }
  void consume(Object x) { ... }
}

class Setup {
  void main() {
    BlockingQueue q = new SomeQueueImplementation();
    Producer p = new Producer(q);
    Consumer c1 = new Consumer(q);
    Consumer c2 = new Consumer(q);
    new Thread(p).start();
    new Thread(c1).start();
    new Thread(c2).start();
  }
}

```

### ArrayBlockingQueue
1、入队列就将尾索引往右移动一个，新元素加入尾索引的位置；
2、出队列就将头索引往尾索引方向移动一个，同时将旧头索引元素设为null，返回旧头索引的元素。
3、一旦数组已满，那么就不允许添加新元素（除非扩充容量）
4、如果尾索引移到了数组的最后（最大索引处），那么就从索引0开始，形成一个“闭合”的数组。
5、由于头索引和尾索引之间的元素都不能为空（因为为空不知道take出来的元素为空还是队列为空），所以删除一个头索引和尾索引之间的元素的话，需要移动删除索引前面或者后面的所有元素，以便填充删除索引的位置。
6、由于是阻塞队列，那么显然需要一个锁，另外由于只是一份数据（一个数组），所以只能有一个锁，也就是同时只能有一个线程操作队列。

主要通过ReentrantLock和Condition实现加锁、阻塞的功能
```java
/** Main lock guarding all access */
final ReentrantLock lock;
/** Condition for waiting takes */
private final Condition notEmpty;
/** Condition for waiting puts */
private final Condition notFull;
/** The queued items */
final Object[] items;
```
put方法，先加锁，这里就证实了它并不是一个高性能容器，因为这里毫无分析的就加锁，性能一定不高
```java
public void put(E e) throws InterruptedException {
    checkNotNull(e);
    //加锁
    final ReentrantLock lock = this.lock;
    //响应阻塞
    lock.lockInterruptibly();
    try {
        while (count == items.length)
            //队列满就等待
            notFull.await();
        enqueue(e);
    } finally {
      //释放锁
        lock.unlock();
    }
}
/**
* Inserts element at current put position, advances, and signals.
* Call only when holding lock.
*/
private void enqueue(E x) {
    // assert lock.getHoldCount() == 1;
    // assert items[putIndex] == null;
    final Object[] items = this.items;
    items[putIndex] = x;
    if (++putIndex == items.length)
        putIndex = 0;
    count++;
    //通知不空
    notEmpty.signal();
}
```
在java中可以使用BlockingQueue来实现消息队列，但是效率不是非常高，因为其内部不是无锁方式。

#### Disruptor
Disruptor可以实现高性能生产者和消费者模式
Disruptor通过以下设计来解决队列速度慢的问题：

- 环形数组结构
为了避免垃圾回收，采用数组而非链表。同时，数组对处理器的缓存机制更加友好。

- 元素位置定位
数组长度2^n，通过位运算，加快定位的速度。下标采取递增的形式。不用担心index溢出的问题。index是long类型，即使100万QPS的处理速度，也需要30万年才能用完。

- 无锁设计
每个生产者或者消费者线程，会先申请可以操作的元素在数组中的位置，申请到之后，直接在该位置写入或者读取数据。

Diruptor 页面：https://github.com/LMAX-Exchange/disruptor
待完成
https://zhuanlan.zhihu.com/p/21355046
https://www.bittiger.io/classpage/QHkP5QobvhNWGZv9f
http://tech.meituan.com/disruptor.html

### LinkedBlockingQueue
LinkedBlockingQueue有两个ReentrantLock和两个Condition以及用于AtomicInteger的count
```java
/** Current number of elements */
private final AtomicInteger count = new AtomicInteger();
/** Lock held by take, poll, etc */
private final ReentrantLock takeLock = new ReentrantLock();
/** Lock held by put, offer, etc */
private final ReentrantLock putLock = new ReentrantLock();
/** Wait queue for waiting takes */
private final Condition notEmpty = takeLock.newCondition();
/** Wait queue for waiting puts */
private final Condition notFull = putLock.newCondition();
/** Linked list node class */
static class Node<E> {
    E item;
    Node<E> next;
    Node(E x) { item = x; }
}
```
但是整体上讲，LinkedBlockingQueue和ConcurrentLinkedQueue的结构类似，都是采用头尾节点，每个节点指向下一个节点的结构，这表示它们在操作上应该类似。
1、LinkedBlockingQueue引入了AtomicInteger的count，这意味着获取队列大小size()已经是常量时间了，不再需要遍历队列。每次队列长度有变更时只需要修改count即可。
2、有了修改Node指向有了锁，所以不需要volatile特性了。
3、引入了两个锁，一个入队列锁，一个出队列锁。当然同时有一个队列不满的Condition和一个队列不空的Condition。

参照锁机制的生产者-消费者模型就知道，入队列就代表生产者，出队列就代表消费者。
为什么需要两个锁？一个锁行不行？其实一个锁完全可以，但是一个锁意味着入队列和出队列同时只能有一个在进行，另一个必须等待其释放锁。
而从ConcurrentLinkedQueue的实现原理来看，事实上head和last (ConcurrentLinkedQueue中是tail)是分离的，互相独立的，这意味着入队列实际上是不会修改出队列的数据的，同时出队列也不会修改入队列，也就是说这两个操作是互不干扰的。
更通俗的将，这个锁相当于两个写入锁，入队列是一种写操作，操作head，出队列是一种写操作，操作tail。可见它们是无关的。但是并非完全无关，后面详细分析。

入队列的阻塞过程大概是这样的：
获取入队列的锁putLock，检测队列大小，如果队列已满，那么就挂起线程，等待队列不满信号notFull的唤醒。
将元素加入到队列尾部，同时修改队列尾部引用last。
队列大小加1。
释放锁putLock。
唤醒notEmpty线程（如果有挂起的出队列线程），告诉消费者，已经有了新的产品。
```java
public void put(E e) throws InterruptedException {
    if (e == null) throw new NullPointerException();
    // Note: convention in all put/take/etc is to preset local var
    // holding count negative to indicate failure unless set.
    int c = -1;
    Node<E> node = new Node<E>(e);
    final ReentrantLock putLock = this.putLock;
    final AtomicInteger count = this.count;
    putLock.lockInterruptibly();
    try {
        /*
         * Note that count is used in wait guard even though it is
         * not protected by lock. This works because count can
         * only decrease at this point (all other puts are shut
         * out by lock), and we (or some other waiting put) are
         * signalled if it ever changes from capacity. Similarly
         * for all other uses of count in other wait guards.
         */
        while (count.get() == capacity) {
            notFull.await();
        }
        enqueue(node);
        c = count.getAndIncrement();
        if (c + 1 < capacity)
            notFull.signal();
    } finally {
        putLock.unlock();
    }
    if (c == 0)
        signalNotEmpty();
}
```

对比入队列，出队列的阻塞过程大概是这样的：

获取出队列的锁takeLock，检测队列大小，如果队列为空，那么就挂起线程，等待队列不为空notEmpty的唤醒。
将元素从头部移除，同时修改队列头部引用head。
队列大小减1。
释放锁takeLock。
唤醒notFull线程（如果有挂起的入队列线程），告诉生产者，现在还有空闲的空间。
## ConcurrentLinkedQueue
>一个基于链接节点的无界线程安全队列。此队列按照 FIFO（先进先出）原则对元素进行排序。
队列的头部 是队列中时间最长的元素。
队列的尾部 是队列中时间最短的元素。
新的元素插入到队列的尾部，队列获取操作从队列头部获得元素。
当多个线程共享访问一个公共 collection 时，ConcurrentLinkedQueue 是一个恰当的选择。此队列不允许使用 null 元素。

主要使用cas原理
ConcurrentLinkedQueue只有头结点、尾节点两个元素，而对于一个节点Node而言除了保存队列元素item外，还有一个指向下一个节点的引用next。
```java
private transient volatile Node<E> head;
private transient volatile Node<E> tail;
private static class Node<E> {
      volatile E item;
      volatile Node<E> next;
      ...
}
```

1、所有结构（head/tail/item/next）都是volatile类型。 这是因为ConcurrentLinkedQueue是非阻塞的，所以只有volatile才能使变量的写操作对后续读操作是可见的（这个是有happens-before法则保证的）。同样也不会导致指令的重排序。
2、由于队列中任何一个节点（Node）只有下一个节点的引用，所以这个队列是单向的，根据FIFO特性，也就是说出队列在头部(head)，入队列在尾部(tail)。头部保存有进入队列最长时间的元素，尾部是最近进入的元素。
3、没有对队列长度进行计数，所以队列的长度是无限的，同时获取队列的长度的时间不是固定的，这**需要遍历整个队列，并且这个计数也可能是不精确的**。
4、初始情况下队列头和队列尾都指向一个空节点，但是非null，这是为了方便操作，不需要每次去判断head/tail是否为空。但是head却不作为存取元素的节点，tail在不等于head情况下保存一个节点元素。也就是说head.item这个应该一直是空，但是tail.item却不一定是空（如果head!=tail，那么tail.item!=null）。
```
public boolean offer(E e) {
    checkNotNull(e);
    final Node<E> newNode = new Node<E>(e);

    for (Node<E> t = tail, p = t;;) {
        Node<E> q = p.next;
        if (q == null) {
            // p is last node
            if (p.casNext(null, newNode)) {
                // Successful CAS is the linearization point
                // for e to become an element of this queue,
                // and for newNode to become "live".
                if (p != t) // hop two nodes at a time
                    casTail(t, newNode);  // Failure is OK.
                return true;
            }
            // Lost CAS race to another thread; re-read next
        }
        else if (p == q)
            // We have fallen off list.  If tail is unchanged, it
            // will also be off-list, in which case we need to
            // jump to head, from which all live nodes are always
            // reachable.  Else the new tail is a better bet.
            p = (t != (t = tail)) ? t : head;
        else
            // Check for tail updates after two hops.
            p = (p != t && t != (t = tail)) ? t : q;
    }
}
```
http://www.infoq.com/cn/articles/ConcurrentLinkedQueue
http://www.blogjava.net/xylz/archive/2010/07/23/326934.html

## 常见的BlockingQueue

![各种](http://7xilc8.com1.z0.glb.clouddn.com/allblockingqueue.png)

## CopyOnWrite容器
Copy-On-Write简称COW，是一种用于程序设计中的优化策略。
其基本思路是，从一开始大家都在共享同一个内容，当某个人想要修改这个内容的时候，才会真正把内容Copy出去形成一个新的内容然后再改，这是一种延时懒惰策略。
从JDK1.5开始Java并发包里提供了两个使用CopyOnWrite机制实现的并发容器,它们是`CopyOnWriteArrayList`和`CopyOnWriteArraySet`。
CopyOnWrite容器非常有用，可以在非常多的并发场景中使用到。

CopyOnWrite容器即写时复制的容器。
通俗的理解是当我们往一个容器添加元素的时候，不直接往当前容器添加，而是先将当前容器进行Copy，复制出一个新的容器，然后新的容器里添加元素，添加完元素之后，再将原容器的引用指向新的容器。
这样做的好处是我们可以对CopyOnWrite容器进行并发的读，而不需要加锁，因为当前容器不会添加任何元素。所以CopyOnWrite容器也是一种读写分离的思想，读和写不同的容器。


### CopyOnWriteArrayList的实现原理
```java
public class CopyOnWriteArrayList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
  /** The lock protecting all mutators */
  final transient ReentrantLock lock = new ReentrantLock();

  /** The array, accessed only via getArray/setArray. */
  private transient volatile Object[] array;
  ...
}
```
ArrayList里添加元素，可以发现在添加的时候是需要加锁的，否则多线程写的时候会Copy出N个副本出来。
```java
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        newElements[len] = e;
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}
```
读的时候不需要加锁，如果读的时候有多个线程正在向ArrayList添加数据，读还是会读到旧的数据，因为写的时候不会锁住旧的ArrayList。
```java
public E get(int index) {
         return (E) a[index];
}
```
### 使用场景
CopyOnWrite的应用场景

CopyOnWrite并发容器用于读多写少的并发场景。比如白名单，黑名单，商品类目的访问和更新场景，假如我们有一个搜索网站，用户在这个网站的搜索框中，输入关键字搜索内容，但是某些关键字不允许被搜索。这些不能被搜索的关键字会被放在一个黑名单当中，黑名单每天晚上更新一次。当用户搜索时，会检查当前关键字在不在黑名单当中，如果在，则提示不能搜索。

使用CopyOnWriteMap需要注意两件事情：
1.减少扩容开销。根据实际需要，初始化CopyOnWriteMap的大小，避免写时CopyOnWriteMap扩容的开销。
2.使用批量添加。因为每次添加，容器每次都会进行复制，所以减少添加次数，可以减少容器的复制次数。
### 缺点
CopyOnWrite容器有很多优点，但是同时也存在两个问题，即内存占用问题和数据一致性问题。所以在开发的时候需要注意一下。

内存占用问题。
因为CopyOnWrite的写时复制机制，所以在进行写操作的时候，内存里会同时驻扎两个对象的内存，旧的对象和新写入的对象（注意:在复制的时候只是复制容器里的引用，只是在写的时候会创建新对象添加到新容器里，而旧容器的对象还在使用，所以有两份对象内存）。
如果这些对象占用的内存比较大，比如说200M左右，那么再写入100M数据进去，内存就会占用300M，那么这个时候很有可能造成频繁的Yong GC和Full GC。

针对内存占用问题，可以通过压缩容器中的元素的方法来减少大对象的内存消耗，比如，如果元素全是10进制的数字，可以考虑把它压缩成36进制或64进制。或者不使用CopyOnWrite容器，而使用其他的并发容器，如ConcurrentHashMap。

数据一致性问题。
CopyOnWrite容器只能保证数据的最终一致性，不能保证数据的实时一致性。所以如果你希望写入的的数据，马上能读到，请不要使用CopyOnWrite容器。

http://ifeve.com/java-copy-on-write/


## volatile final
不可变一定是线程安全的
## 同步更新、互斥同步、非阻塞同步
