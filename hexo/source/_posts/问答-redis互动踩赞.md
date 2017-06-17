title: 问答-redis互动踩赞
toc: true
date: 2017-02-27 14:45:18
tags: wenda
categories: note
---
Redis是一个开源，高级的键值存储和一个适用的解决方案，用于构建高性能，可扩展的Web应用程序。
传统数据库中经常要查询最新的、最热的，很可能因为访问请求大的时候写硬盘可能导致锁竞争从而导致程序阻塞变慢，而redis就能解决这个问题

Redis有三个主要特点，使它优越于其它键值数据存储系统

- Redis将其数据库完全保存在内存中，仅使用磁盘进行持久化。
- 与其它键值数据存储相比，Redis有一组相对丰富的数据类型。
- Redis可以将数据复制到任意数量的从机中。

以下是Redis的一些优点。

- 异常快
Redis非常快，每秒可执行大约110000次的设置(SET)操作，每秒大约可执行81000次的读取/获取(GET)操作。
- 支持丰富的数据类型
Redis支持开发人员常用的大多数数据类型，例如列表，集合，排序集和散列等等。这使得Redis很容易被用来解决各种问题，因为我们知道哪些问题可以更好使用地哪些数据类型来处理解决。
- 操作具有原子性
所有Redis操作都是原子操作，这确保如果两个客户端并发访问，Redis服务器能接收更新的值。
- 多实用工具
Redis是一个多实用工具，可用于多种用例，如：缓存，消息队列(Redis本地支持发布/订阅)，应用程序中的任何短期数据，例如，web应用程序中的会话，网页命中计数等。



缺点

1. Redis不具备自动容错和恢复功能，主机从机的宕机都会导致前端部分读写请求失败，需要等待机器重启或者手动切换前端的IP才能恢复。

2. 主机宕机，宕机前有部分数据未能及时同步到从机，切换IP后还会引入数据不一致的问题，降低了系统的可用性。

3. Redis的主从复制采用全量复制，复制过程中主机会fork出一个子进程对内存做一份快照，并将子进程的内存快照保存为文件发送给从机，这一过程需要确保主机有足够多的空余内存。若快照文件较大，对集群的服务能力会产生较大的影响，而且复制过程是在从机新加入集群或者从机和主机网络断开重连时都会进行，也就是网络波动都会造成主机和从机间的一次全量的数据复制，这对实际的系统运营造成了不小的麻烦。

4. Redis较难支持在线扩容，在集群容量达到上限时在线扩容会变得很复杂。为避免这一问题，运维人员在系统上线时必须确保有足够的空间，这对资源造成了很大的浪费。

http://www.yiibai.com/redis/
https://redis.io/

java中实现redis需要导包
```
<dependency>
	<groupId>redis.clients</groupId>
	<artifactId>jedis</artifactId>
	<version>2.8.0</version>
</dependency>
```


## redis使用


其中主要的数据类型有
- 字符串

- hashes
hash表
https://redis.io/commands#hash
![sorted sets](http://7xilc8.com1.z0.glb.clouddn.com/rhash.png)
- lists
双向链表，其中bp开头的可是实现阻塞的功能
https://redis.io/commands#list
![lists](http://7xilc8.com1.z0.glb.clouddn.com/rlist.png)
- sets
集，不允许元素重复,同时还可以做一些运算，比如交集、并集、补集等
https://redis.io/commands#set
![sets](http://7xilc8.com1.z0.glb.clouddn.com/rset.png)

- sorted sets
有序集，每个点都有自己的分数，用来排序
https://redis.io/commands#sorted_set
![sorted sets](http://7xilc8.com1.z0.glb.clouddn.com/rsortset.png)



```
//选择数据库，一共是1-16个
select 6
//查找所有的键值，可以用正则表达式
keys *
```

### sorted set数据结构
在java中实现sortedset使用的是红黑树，redis使用了跳表，查询、插入、删除时间复杂度期望值O（logn）

分数，节点，表头


http://redisbook.com/preview/skiplist/datastruct.html
http://blog.csdn.net/ict2014/article/details/17394259
http://www.leoox.com/?p=347


### java中调用redis
需要建立一个redisAdaptor来开启redis，封装各种命令
```
@Service
public class JedisAdapter implements InitializingBean {
    private static final Logger logger = LoggerFactory.getLogger(JedisAdapter.class);
    private JedisPool pool;
    @Override
    public void afterPropertiesSet() throws Exception {
        pool = new JedisPool("redis://localhost:6379/10");
    }

    public Jedis getJedis() {
       return pool.getResource();
    }

    //开启线程
    public Transaction multi(Jedis jedis) {
        try {
            return jedis.multi();
        } catch (Exception e) {
            logger.error("发生异常" + e.getMessage());
        } finally {
        }
        return null;
    }

		//执行事务
		public List<Object> exec(Transaction tx, Jedis jedis) {
				try {
						return tx.exec();
				} catch (Exception e) {
						logger.error("发生异常" + e.getMessage());
						tx.discard();
				} finally {
						if (tx != null) {
								try {
										tx.close();
								} catch (IOException ioe) {
										// ..
								}
						}

						if (jedis != null) {
								jedis.close();
						}
				}
				return null;
		}


    //封装，因为这里面要用到关闭资源等
    public long sadd(String key, String value) {
        Jedis jedis = null;
        try {
            jedis = pool.getResource();
            return jedis.sadd(key, value);
        } catch (Exception e) {
            logger.error("发生异常" + e.getMessage());
        } finally {
            if (jedis != null) {
                jedis.close();
            }
        }
        return 0;
    }

}
```


### redis持久化
- RDB 持久化可以在指定的时间间隔内生成数据集的时间点快照（point-in-time snapshot）。
- AOF 持久化记录服务器执行的所有写操作命令，并在服务器启动时，通过重新执行这些命令来还原数据集。
AOF 文件中的命令全部以 Redis 协议的格式来保存，新命令会被追加到文件的末尾。 Redis 还可以在后台对 AOF 文件进行**重写（rewrite）** ，使得 AOF 文件的体积不会超出保存数据集状态所需的实际大小。
- Redis 还可以同时使用 AOF 持久化和 RDB 持久化。
在这种情况下， 当 Redis 重启时， 它会优先使用 AOF 文件来还原数据集， 因为 AOF 文件保存的数据集通常比 RDB 文件所保存的数据集更完整。

你甚至可以关闭持久化功能，让数据只在服务器运行时存在。


#### RDB 的优点

**RDB 是一个非常紧凑（compact）的文件，它保存了 Redis 在某个时间点上的数据集。** 这种文件非常适合用于进行备份： 比如说，你可以在最近的 24 小时内，每小时备份一次 RDB 文件，并且在每个月的每一天，也备份一个 RDB 文件。 这样的话，即使遇上问题，也可以随时将数据集还原到不同的版本。

RDB 非常适用于**灾难恢复（disaster recovery）** ：它只有一个文件，并且内容都非常紧凑，可以（在加密后）将它传送到别的数据中心，或者亚马逊 S3 中。

RDB 可以最大化 Redis 的性能：父进程在保存 RDB 文件时唯一要做的就是 fork 出一个子进程，然后这个子进程就会处理接下来的所有保存工作，父进程无须执行任何磁盘 I/O 操作。

RDB 在恢复大数据集时的速度比 AOF 的恢复速度要快。

#### RDB 的缺点
**如果你需要尽量避免在服务器故障时丢失数据，那么 RDB 不适合你。** 虽然 Redis 允许你设置不同的保存点（save point）来控制保存 RDB 文件的频率， 但是， 因为RDB 文件需要保存整个数据集的状态， 所以它并不是一个轻松的操作。 因此你可能会至少 5 分钟才保存一次 RDB 文件。 在这种情况下， 一旦发生故障停机， 你就可能会丢失好几分钟的数据。

每次保存 RDB 的时候，Redis 都要 fork() 出一个子进程，并由子进程来进行实际的持久化工作。 在数据集比较庞大时， fork() 可能会非常耗时，造成服务器在某某毫秒内停止处理客户端； 如果数据集非常巨大，并且 CPU 时间非常紧张的话，那么这种停止时间甚至可能会长达整整一秒。 虽然 AOF 重写也需要进行 fork() ，但无论 AOF 重写的执行间隔有多长，数据的耐久性都不会有任何损失。


#### AOF 的优点
使用 AOF 持久化会让 Redis 变得非常耐久（much more durable）：你可以设置不同的 fsync 策略，比如无 fsync ，每秒钟一次 fsync ，或者每次执行写入命令时 fsync 。 AOF 的默认策略为每秒钟 fsync 一次，在这种配置下，Redis 仍然可以保持良好的性能，并且就算发生故障停机，也最多只会丢失一秒钟的数据（ fsync 会在后台线程执行，所以主线程可以继续努力地处理命令请求）。

AOF 文件是一个只进行追加操作的日志文件（append only log）， 因此对 AOF 文件的写入不需要进行 seek ， 即使日志因为某些原因而包含了未写入完整的命令（比如写入时磁盘已满，写入中途停机，等等）， redis-check-aof 工具也可以轻易地修复这种问题。

Redis 可以在 AOF 文件体积变得过大时，自动地在后台对 AOF 进行重写： 重写后的新 AOF 文件包含了恢复当前数据集所需的最小命令集合。 整个重写操作是绝对安全的，因为 Redis 在创建新 AOF 文件的过程中，会继续将命令追加到现有的 AOF 文件里面，即使重写过程中发生停机，现有的 AOF 文件也不会丢失。 而一旦新 AOF 文件创建完毕，Redis 就会从旧 AOF 文件切换到新 AOF 文件，并开始对新 AOF 文件进行追加操作。
因为 AOF 的运作方式是不断地将命令追加到文件的末尾， 所以随着写入命令的不断增加， AOF 文件的体积也会变得越来越大。
举个例子， 如果你对一个计数器调用了 100 次 INCR ， 那么仅仅是为了保存这个计数器的当前值， AOF 文件就需要使用 100 条记录（entry）。
然而在实际上， 只使用一条 SET 命令已经足以保存计数器的当前值了， 其余 99 条记录实际上都是多余的。
为了处理这种情况， Redis 支持一种有趣的特性： 可以在不打断服务客户端的情况下， 对 AOF 文件进行重建（rebuild）。
执行 BGREWRITEAOF 命令， **Redis 将生成一个新的 AOF 文件， 这个文件包含重建当前数据集所需的最少命令。**

AOF 文件有序地保存了对数据库执行的所有写入操作， 这些写入操作以 Redis 协议的格式保存， 因此 AOF 文件的内容非常容易被人读懂， 对文件进行分析（parse）也很轻松。 导出（export） AOF 文件也非常简单： 举个例子， 如果你不小心执行了 FLUSHALL 命令， 但只要 AOF 文件未被重写， 那么只要停止服务器， 移除 AOF 文件末尾的 FLUSHALL 命令， 并重启 Redis ， 就可以将数据集恢复到 FLUSHALL 执行之前的状态。

#### AOF 的缺点
对于相同的数据集来说，AOF 文件的体积通常要大于 RDB 文件的体积。
根据所使用的 fsync 策略，AOF 的速度可能会慢于 RDB 。 在一般情况下， 每秒 fsync 的性能依然非常高， 而关闭 fsync 可以让 AOF 的速度和 RDB 一样快， 即使在高负荷之下也是如此。 不过在处理巨大的写入载入时，RDB 可以提供更有保证的最大延迟时间（latency）。
AOF 在过去曾经发生过这样的 bug ： 因为个别命令的原因，导致 AOF 文件在重新载入时，无法将数据集恢复成保存时的原样。 （举个例子，阻塞命令 BRPOPLPUSH 就曾经引起过这样的 bug 。） 测试套件里为这种情况添加了测试： 它们会自动生成随机的、复杂的数据集， 并通过重新载入这些数据来确保一切正常。 虽然这种 bug 在 AOF 文件中并不常见， 但是对比来说， RDB 几乎是不可能出现这种 bug 的。

http://doc.redisfans.com/topic/persistence.html

## 踩赞的实现
基本原理是将踩赞等记录存到redis中，redis使用`set`存储实体与点赞人的关系
key = LIKE:实体类型:实体id
value = 用户id

比如用户2点赞了评论(实体类型是1)5

- 添加赞
`sadd LIKE:1:5 2`
同时还要去掉踩的记录
`srem DISLIKE:1:5:2`
Java代码实现
```
//事务
Jedis jedis = jedisAdapter.getJedis();
Transaction tx = jedisAdapter.multi(jedis);
jedisAdapter.sadd(likeKey, String.valueOf(userId));
jedisAdapter.srem(disLikeKey, String.valueOf(userId));
List<Object> ret = jedisAdapter.exec(tx, jedis);
```

- 取出喜欢的记录
`smembers LIKE:2:1`

- 统计喜欢的记录数
`scard LIKE:2:1`


## 互粉的实现
原理和踩赞的实现相似，但是粉丝使用sorted set存储数据，因为涉及到排序

key = FOLLOWER:实体类型:实体id
value = 用户id

比如用户2关注了当前用户(实体类型是1)5
- 添加关注
要同时添加两个实体的集合，所以加上事务
zadd 关键字 分数 值，这里分数是用来排序的
实体粉丝关注当前用户
`zadd FOLLOWER:1:5 9 2`分数是9
当前用户对实体关注加1
`zadd FOLLOWEE:2:1 9 5`

- 查看关注者
zrange 关键字 位置1 位置2，zrange是查看位置的元素，即从第0个到第-1个，-1表示倒数第一个
`zrange FOLLOWER:1:5 0 -1 withscores`
zrevrange 关键字 位置1 位置2，倒序查看
`zrange FOLLOWER:1:5 0 10 withscores`
如果以时间为分数，那么反向查看就是查看最新的

- 得到粉丝个数
zcard 关键字
`zecard FOLLOWER:1:5`

- 取消关注
要同时取消双方的，且为一个事务
`zrem FOLLOWER:1:5 1`

- 找出几个用户最近关注的10个问题
ZUNIONSTORE 求并集
http://www.runoob.com/redis/sorted-sets-zunionstore.html

- 找出两个人共同关注的
ZINTERSTORE 求交集
http://www.runoob.com/redis/sorted-sets-zinterstore.html
