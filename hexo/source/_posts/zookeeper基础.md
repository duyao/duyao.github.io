title: zookeeper基础
toc: true
date: 2017-05-28 16:40:02
tags: zookeeper
categories: note
---
# zk介绍
>Zookeeper是Apache Hadoop的一个子项目，本质是一个分布式小文件系统，主要解决分布式系统应用中遇到的数据管理问题（一致性问题）：统一命名服务，状态同步服务，集群管理，分布式应用配置项的管理。

## 基本命令
| 命令 | 作用 |
| :-------------: | :-------------: |
| stat path [watch] | f |
| set path data [version] | stat |
| ls path [watch] | stat |
| delquota [-n/-b] path | stat |
| ls2 path [watch] | stat |
| setAcl path acl | stat |
| setquota -n/-b val path | stat |
| history | stat |
| redo cmdno | stat |
| printwatches on/off | stat |
| delete path [version] | stat |
| sync path | stat |
| listquota path | stat |
| rmr path | stat |
| get path [watch] | stat |
| create [-s] [-e] path data acl | stat |
| addauth scheme auth | stat |
| quit | stat |
| getAcl path | stat |
| close | stat |
| connect host:port | stat |

## 数据模型
![zk中的数据结构](http://7xilc8.com1.z0.glb.clouddn.com/zk.png)
1. 每个子目录项（比如图中的NameService）都被称为znode。
znode可以有子节点目录，并且每个znode都可以保存数据，但注意`EPHEMERAL`类型的目录节点不能有子节点目录。
2. znode是有版本的，每个znode中存储的数据可以有多个版本，也就是同一访问路径可以存储多份数据。
3. 对于临时的znode，一旦创建这个znode的客户端与服务器失去联系，这个znode也将自动删除，Zookeeper的客户端和服务器通信采用长链接方式，每个客户端和服务器通过心跳来保持连接。
4. znode可以被监控，包括这个节点中存储的数据的修改，子节点目录的变化等，一旦变化可以通知监听的客户端，这个是Zookeeper的重要特性。

## 节点类型
Zookeeper节点在创建后其类型不能被修改，有四种类型的节点:
- `PERSISTENT`
所谓持久节点，是指在节点创建后，就一直存在，直到有删除操作来主动清除这个节点——不会因为创建该节点的客户端会话失效而消失。
- `PERSISTENT_SEQUENTIAL`
这类节点的基本特性和上面的节点类型是一致的。额外的特性是，在ZK中，每个父节点会为他的第一级子节点维护一份时序，会记录每个子节点创建的先后顺序。基于这个特性，在创建子节点的时候，可以设置这个属性，那么在创建节点过程中，ZK会自动为给定节点名加上一个数字后缀，作为新的节点名。这个数字后缀的上限是整型的最大值。
- `EPHEMERAL`
和持久节点不同的是，临时节点的生命周期和客户端会话绑定。也就是说，如果客户端会话失效，那么这个节点就会自动被清除掉。注意，这里提到的是会话失效，而非连接断开。另外，在临时节点下面不能创建子节点。
- `EPHEMERAL_SEQUENTIAL`
ZK会自动为给定节点名的临时节点加上一个数字后缀，作为新的节点名。同持久顺序节点。

## 节点信息
zookeeper目录中的每一个节点对应着一个znode，每个znode维护着一个属性结构，它包含数据的版本号、时间戳、等信息，同时每个Znode由3部分组成:
① stat：此为状态信息, 描述该Znode的版本, 权限等信息
② data：与该Znode关联的数据
③ children：该Znode下的子节点

Zookeeper就是通过这些属性来实现它特定的功能。每当znode的数据改变时，相应的版本号会增加，每当客户端查询、更新和删除数据时，也必须提供要被操作的znode版本号，如果所提供的数据版本号与实际的不匹配，那么将会操作失败。
节点属性：
```
[zk: localhost:2181(CONNECTED) 2] get /test
123
cZxid = 0x6
ctime = Sun May 28 15:24:48 CST 2017
mZxid = 0x6
mtime = Sun May 28 15:24:48 CST 2017
pZxid = 0x6
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 3
numChildren = 0
```
| 属性 | 描述  |
| :-------------: | :-------------: |
| cZxid | 节点被创建的zxid值  |
| ctime | 节点创建的时间  |
| mZxid | 节点被修改时zxid值  |
| mtime | 节点最后一次的修改时间  |
| pZxid | 最后一次更改该节点孩子的事务id  |
| cvesion | 节点所拥有的子节点被修改的版本号  |
| dataVersion | 数据版本  |
| aclVersion | 访问控制版本  |
| ephemeralOwner | 如果节点为临时节点，那么它的值为这个节点拥有者的session ID；负责它的值为0  |
| dataLength | 节点数据的长度  |
| numChildren | 节点拥有子节点的个数  |


## watcher
### watcher监听器
ZooKeeper中所有的读操作`getData()`,` getChildren()`和 `exists()`可以选择设置一个watcher。
还有在创建客户端对象实例时也可以设置watcher，`ZooKeeper(String connectString, int sessionTimeout, Watcher watcher)`
ZooKeeper有两个监听器列表：数据监听和子节点监听:`getData()`和`exists()`设置数据监听器; `getChildren()`设置子节点监听器。
只能二选一，根据返回数据的类型来设置监听器。`getData()`和`exists()`返回节点的数据信息，然而`getChildren()`返回一个子节点列表。
![设置监视器的操作及对应的触发器](http://7xilc8.com1.z0.glb.clouddn.com/watcheraction.png)
因此，setData()会触发数据监听器。一个成功的 create()会触发一个数据监听器。一个delete()会触发数据监听器和子节点监听器。
![wathcer的触发及改变](http://7xilc8.com1.z0.glb.clouddn.com/watcherchanged.png)
### 注意点
1.**Watches通知是一次性的**，必须重复注册.
2.同一个ZK客户端，反复对同一个ZK节点（znode）注册相同的watcher，是无效的，**最终只会有一个生效**。
3.发生`CONNECTIONLOSS`之后，只要在`session_timeout`之内再次连接上（即不发生`SESSIONEXPIRED`），那么这个连接注册的watches依然在。
4.客户端会话失效之后，所有这个会话中创建的Watcher都会被移除。
5.节点数据的版本变化会触发`NodeDataChanged`，注意，这里特意说明了是版本变化。存在这样的情况，**只要成功执行了setData()方法，无论内容是否和之前一致，都会触发NodeDataChanged事件**。
6.对某个节点注册了watcher，但是**节点被删除了，那么注册在这个节点上的watcher都会被移除**。
### 注册过程
总体来说.可以概括为以下几个过程:
（1）客户端注册Watcher
（2）服务端处理Watcher
（3）客户端回调Watcher
首先需要了解一下zookeeper中的watchedevent结构，其包括三个基本属性：通知状态（keepstate）、事件类型（eventType）和节点路径（path）。
zookeeper使用watchedevent对象来封装服务端事件并传递给watcher，而实际传递的是watcherevent，其结构和watchedevent一样，
`WatcherEvent` 实体实现了序列化接口，因此可以用于网络传输。数据结构如下。
```
Class WatcherEvent{
  type:int
  state:int
  path:String
}
```
从这里可以看出，**watcher事件只是一个简单的事件说明，并不包含事件的数据变更内容**，这样能保证网络传输的高效性。
这里也体现出观察者模式中推拉结合，发生的变化被推送到观察者，此时观察者只知道发生了变化，只有靠拉才能知道到底发生了什么变化。

http://blog.csdn.net/lionaiying/article/details/53915878

`process()` 是 Watch 接口中的回调方法。当 ZooKeeper 向客户端发送一个 Watcher 时间通知时，客户端就会对相应的 process 方法进行回调，从而实现对事件的处理。 like this：syncNodes（）方法。
`abstract public  void process ( WatchedEvent event )`

http://www.cnblogs.com/rocky24/p/4859206.html
https://www.ibm.com/developerworks/cn/opensource/os-cn-apache-zookeeper-watcher/

#### ServerCnxn类及cnxn对象
- Zk客户端与服务器之间的tcp连接
- 实现了watcher接口
总结：既包含了连接信息又包了watcher信息
#### watchManager
- Zk服务器端Watcher的管理者
- 从两个维度维护watcher
• watchTable:从数据节点的粒度来维护
• watch2Paths从watcher的粒度来维护
- 负责watcher事件的触发
## ACL
`Scheme:id:permission`
比如`world:anyone:crdwa`
- Scheme:验证过程中使用的检验策略
- Id:权限被赋予的对象，比如ip或者某个用户
- Permission:权限，Ї面的crdwa，表示五个权限组合
通过`setAcl`命令置节点的权限,`getAcl`可以查看节点的Acl信息
```
[zk: localhost:2181(CONNECTED) 8] setAcl /test world:anyone:ca
cZxid = 0x6
ctime = Sun May 28 15:24:48 CST 2017
mZxid = 0x6
mtime = Sun May 28 15:24:48 CST 2017
pZxid = 0xa
cversion = 1
dataVersion = 0
aclVersion = 1
ephemeralOwner = 0x0
dataLength = 3
numChildren = 1
[zk: localhost:2181(CONNECTED) 9] getAcl /test
'world,'anyone
: ca
```
**节点的acl不具有继承关系**
### Scheme类型
#### world

Id的固定值为anyone，表示任何用户
`world:anyone:crdwa`表示任何用户都Ӏ有crdwa权限
#### auth
`auth:username:password:crdwa`
表示给**认证通过的所有用户**设置acl权限，同时可以添加多个用户
通过addauth命令进行认证用户的添加`addauth digest <username>:<password>`
Auth策略的本质就是digest
如果通过addauth创建多组用户和密码，当使用setAcl修改权限时，所有的用户和密码的权限都会跟着修改
通过addauth新创建的用户和密码组需要重新调用setAcl才会加入到权限组中去
#### digest
`Scheme:id:permission`，比如 `digest:username:password:crdwa`
指定某个用户及它的密码可以访问
这里的`username:password`必须**经过SHA-1和BASE64编码**，即`BASE64(SHA1(username:password))`
通过addauth命令进行认证用户的添加`addauth digest <username>:<password>`
#### IP
`Scheme:id:permission` ，比如 `ip:127.0.0.1:crdwa`
指定某个ip地址可以访问
#### super
供运维人员维护节点使用
有权限操作任何节点
启动时，在命令参数中配置
- `-Dzookeeper.DigestAuthenticationProvider.superDigest=admin:015uTByzA4zSglcmseJsxTo7n3c=`
- 打开zkCli.cmd，在java命令后添加以上配置
用户名和密码也需要通过sha1和base64编码
https://holynull.gitbooks.io/zookeeper/content/cao_zuo_operations.html
# zk用途
# zk原理
Zookeeper集群的特点
- 是一种对等集群，所有节点的数据都一样
- 集群节点之间靠心跳感知彼此的存在
- 所有写操作都在主节点，其他节点只能读，虽然可以接收写请求，但是内部会把写操作转给主节点
- 通过选举机制选出主节点，从而保障了主节点的高可用
- 至少少3个节点，必须是奇数个节点(与选举算法相关)
- 当一半以上的节点数据写入ۨ成功，则返回写入ۨ成功，是最终一致性的策略

## ZAB协议
ZooKeeper Atomic Broadcast即ZooKeeper原子消息广播协议，简称ZAB
- 选举过程需要依赖ZAB协议
- 数据写入过程也需要ZAB协议
- ZAB的核心是定义了那些会改变zk服务器数据状态的请求的处理方式

所有事物请求必须由一个全局唯一的服务器来协调处理，该服务器被称Leader服务器，而剩余的其它服务器则称为Follower服务器。
Leader服务器负责将一个客户端事物请求转换成那个一个事物Proposa，并将该Proposal分发给集群中所有的Follower服务器。
之后Leader服务器需要等待所有Follower服务器的反馈，一旦超过半数的Follower服务器进行了正确的反馈后，那么Leader就会再次向所有的Follower服务器分发Commit消息，要求其将前一个Proposal进行提交。
### 服务器角色
Leader
- 事物请求的唯一调度和处理者，保证集群事物处理的顺序性
- 集群内部各服务器的调度者
Follower
- 处理客户端非事物请求，转发事物请求给Leader服务器
- 参与事物请求Proposal的投票
- 参与Leader选举投票
 Observer
- 处理客户端非事物请求，转发事物请求给Leader服务器
- 不参与任何形式的投票，包括选举和事物投票超过半数确认
- 此角色在通常是为了提高读性能

服务器状态
LOOKING
- 寻找Leader状态
- 当服务器处于此状态时，表示当前没有Leader，需要进入选举流程
FOLLOWING
- 跟随者状态，表明当前服务器角色是Follower
OBSERVING
- 观察者状态，表明当前服务器角色ОObserver
LEADING
- 领导者状态，表明当前服务器角色ОLeader
### ZAB协议三阶段
- 发现Discovery，即选举Leader过程
- 同步Synchronization，选举出新的Leader后，Follwer或者Observer从Leader同步最新的数据
- 广播，同步完成后，就可以接收客户端新的事物请求，并进行消息广播，实现数据在集群节点的副本存储
### 通信
基于TCP协议
- О了避免୍复创建两个节点之间的tcp连接，zk按照myid数值方向来建立连接，即小数的节点发起大的
节点连接，比如idО1的向idО2的发起tcp连接
 多端口
- 配置中第一个端口是通信和数据同࠵端口，默认是2888
- 第и个端口是投票端口，默认是3888

只支持FastLeaderElection的tcp协议版本的选举算法
## 会话管理
# 客户端
Curator介绍
