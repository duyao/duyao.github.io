title: juc之无锁
toc: true
date: 2017-03-01 08:40:21
tags: jvm
categories: note
---

# 无锁
java.util.concurrent.atomic包中的类
## CAS
CAS-compareAndSet算法的过程是这样：它包含3个参数CAS(V,E,N)。V表示要更新的变量，E表示预期值，N表示新值。仅当V值等于E值时，才会将V的值设为N，如果V值和E值不同，则说明已经有其他线程做了更新，则当前线程什么都不做。最后，CAS返回当前V的真实值。
CAS操作是抱着乐观的态度进行的，它总是认为自己可以成功完成操作。当多个线程同时使用CAS操作一个变量时，只有一个会胜出，并成功更新，其余均会失败。失败的线程不会被挂起，仅是被告知失败，并且允许再次尝试，当然也允许失败的线程放弃操作。
基于这样的原理，CAS操作即时没有锁，也可以发现其他线程对当前线程的干扰，并进行恰当的处理。


# 无锁类的使用

## AtomicInteger
继承于`Number`，主要使用处理器提供的CMPXCHG指令实现的
### 主要接口
其内部有一个volatile的value，unsafe是一个底层调用c语言来实现的类，offset是偏移量
```java
private volatile int value;
private static final Unsafe unsafe = Unsafe.getUnsafe();
private static final long valueOffset;
```
所有的操作都是调用cas实现，比较value和期望值来实现的
public final boolean compareAndSet(int expect, int u)//如果当前值为expect，则设置为u

value的get和set方法
public final int get() //取得当前值
public final void set(int newValue) //设置当前值

其余的函数基本是就是调用cas来实现的
public final int getAndSet(int newValue) //设置新值，并返回旧值
public final int getAndIncrement() //当前值加1，返回旧值
public final int getAndDecrement() //当前值减1，返回旧值
public final int getAndAdd(int delta) //当前值增加delta，返回旧值
public final int incrementAndGet() //当前值加1，返回新值
public final int decrementAndGet() //当前值减1，返回新值
public final int addAndGet(int delta) //当前值增加delta，返回新值

### 接口实现


在jdk1.7中大部分的方法都是使用cas函数，在死循环中不停的比较，得出正确结果后退出的
```java
public final int getAndIncrement() {
    for (;;) {
        int current = get();
        int next = current + 1;
        if (compareAndSet(current, next))
            return current;
    }
}
public final boolean compareAndSet(int expect, int update) {
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}
```
在jdk1.8中，使用unsafe实现，jdk1.8中更新了unsafe
```java
public final int getAndIncrement() {
    return unsafe.getAndAddInt(this, valueOffset, 1);
}
```

## unsafe
Java无法直接访问底层操作系统，而是通过本地（native）方法来访问。不过尽管如此，JVM还是开了一个后门，JDK中有一个类Unsafe，它提供了硬件级别的原子操作。
Unsafe类是一个可以执行不安全、容易犯错的操作的一个特殊类。虽然Unsafe类中所有方法都是public的，但是这个类只能在一些被信任的代码中使用。
- 根据偏移量设置值
根据offset得到一个数值的位置，这样就可以设置
- park()
停止操作
- 底层的CAS操作
- 非公开API，在不同版本的JDK中， 可能有较大差异

## AtomicReference
对引用进行修改，是一个模板类，抽象化了数据类型
```java
public static void main(String[] args) {
		AtomicReference<String> string = new AtomicReference<>("abc");

		for (int i = 0; i < 10; i++) {
				new Thread(new Runnable() {
						@Override
						public void run() {
								if (string.compareAndSet("abc", "def")) {
										System.out.println(Thread.currentThread().getId() + "changed");

								} else {
										System.out.println(Thread.currentThread().getId() + "failed");
								}

						}
				}).start();

		}
}
```
## AtomicStampedReference
主要解决了ABA问题，aba是不仅要求结果的正确性还对数据修改的过程敏感。
因为CAS需要在操作值的时候检查下值有没有发生变化，如果没有发生变化则更新。
但是如果一个值原来是A，变成了B，又变成了A，那么使用CAS进行检查时会发现它的值没有发生变化，但是实际上却变化了。
ABA问题的解决思路就是使用版本号。在变量前面追加上版本号，每次变量更新的时候把版本号加一，那么A－B－A 就会变成1A-2B－3A。

实现过程中加入了版本号pair
```java
private static class Pair<T> {
    final T reference;
    final int stamp;
    private Pair(T reference, int stamp) {
        this.reference = reference;
        this.stamp = stamp;
    }
    static <T> Pair<T> of(T reference, int stamp) {
        return new Pair<T>(reference, stamp);
    }
}
```

## AtomicIntegerArray
支持无锁的数组
通过计算每个数字在位置(利用base和shift)该改变数值
```java
private static final Unsafe unsafe = Unsafe.getUnsafe();
private static final int base = unsafe.arrayBaseOffset(int[].class);
private static final int shift;
private final int[] array;
```
### 主要接口
//获得数组第i个下标的元素
public final int get(int i)
//获得数组的长度
public final int length()
//将数组第i个下标设置为newValue，并返回旧的值
public final int getAndSet(int i, int newValue)
//进行CAS操作，如果第i个下标的元素等于expect，则设置为update，设置成功返回true
public final boolean compareAndSet(int i, int expect, int update)
//将第i个下标的元素加1
public final int getAndIncrement(int i)
//将第i个下标的元素减1
public final int getAndDecrement(int i)
//将第i个下标的元素增加delta（delta可以是负数）

## AtomicIntegerFieldUpdater
让普通变量也享受原子操作
### 主要方法
//构造方法
`public static <U> AtomicIntegerFieldUpdater<U> newUpdater(Class<U> tclass, String fieldName)`

其余方法和AtomicInteger类似
### 使用
- Updater只能修改它可见范围内的变量
因为Updater使用反射得到这个变量。如果变量不可见，就会出错。
比如如果变量为private，就是不可行的。

- 必须是volatile类型
为了确保变量被正确的读取，它必须是volatile类型的。
如果我们原有代码中未申明这个类型，那么简单得申明一下就行，这不会引起什么问题

- 不支持static字段
由于CAS操作会通过对象实例中的偏移量直接进行赋值，因此，它不支持static字段（Unsafe.objectFieldOffset()不支持静态变量）。


```java
public class AtomicIntegerFieldUpdaterDemo {
    public static class Student {
        int id;
        //必须为volatile类型
        volatile int score;
    }

    public static void main(String[] args) {
        //构造方法，一个参数是类，第二是参数名
        AtomicIntegerFieldUpdater updater = AtomicIntegerFieldUpdater.
                newUpdater(Student.class, "score");
        Random r = new Random();
        Student stu = new Student();
        //验证结果是否正确
        AtomicInteger comparedInteger = new AtomicInteger();
        Thread[] threads = new Thread[1000];
        for (int i = 0; i < 1000; i++) {
            threads[i] = new Thread(new Runnable() {
                @Override
                public void run() {
                    if (r.nextInt() > 10) {
                        updater.incrementAndGet(stu);
                        comparedInteger.incrementAndGet();

                    }
                }
            });
            threads[i].start();

        }
        //等待所有线程完成
        for (int i = 0; i < 1000; i++) {
            try {
                threads[i].join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        }
        System.out.println("comparedInteger = " + comparedInteger.get());
        System.out.println("stu.score = " + stu.score);


    }
}
```
# 其他问题
## Long、AtomicLong、LongAdder
在java中对于long的操作不是原子性，尤其是32位操作系统，因为long是64bit的

因此要使用AtomicLong，涉及并发的地方都是使用CAS操作，在硬件层次上去做 compare and set操作。效率非常高。
AtomicLong的实现方式是内部有个value 变量，当多线程并发自增，自减时，均通过CAS 指令从机器指令级别操作保证并发的原子性

LongAdder是java8中新添加的类
唯一会制约AtomicLong高效的原因是高并发，高并发意味着CAS的失败几率更高， 重试次数更多，越多线程重试，CAS失败几率又越高，变成恶性循环，AtomicLong效率降低。
因此在LongAdder中，在低并发时，还是CAS操作，因为低并发时，casBase操作基本都会成功，只有并发高到一定程度了，才会进入分支中分段更新的操作，将单一value的更新压力分担到多个value中去，降低单个value的 “热度”，这样就减少并发。
**低并发时LongAdder和AtomicLong性能差不多，高并发时LongAdder更高效！**

http://coolshell.cn/articles/11454.html

## 无锁的Vector实现
待完成
