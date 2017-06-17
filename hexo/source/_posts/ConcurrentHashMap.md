title: ConcurrentHashMap
toc: true
date: 2016-03-25 13:43:29
tags: java
categories:
---

ConcurrentHashMap是Java 5中支持高并发、 高吞吐的HashMap。
它的关键技术是：锁分离技术。 它使用了多个锁来控制对hash表的不同部分进行修改。

## 实现

ConcurrentHashMap包含了两个静态内部类：

- HashEntry
用来封装映射表的键值对
- Segment
用来充当锁的角色， 是一种可重入锁ReentrantLock 利用了锁分离技术来保护不同的segment

ConcurrentHashMap中的Segment就管理若干个的HashTable， 每个HashTable由多个HashEntry组成。 每个Segment持有自己的锁， 只要修改操作发生在不同的Segment上， 就可以并发执行

### 具体实现
每个Segment对象守护整个散列表的若干个桶，每个桶由若干个HashEntry对象连接起来的

### ConcurrentHashMap是弱一致的

ConcurrentHashMap进行操作时， put操作将一个元素加入到底层数据结构后，get可能在某段时间内还看不到这个元素。
ConcurrentHashMap的弱一致性主要是为了提升效率， 是一致性与效率之间的一种权衡。
要成为强一致性， 就得到处使用锁， 甚至是全局锁， 这就与Hashtable和同步的HashMap一样了


## 深度好文
[探索 ConcurrentHashMap 高并发性的实现机制](https://www.ibm.com/developerworks/cn/java/java-lo-concurrenthashmap/)
