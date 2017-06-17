title: Hadoop安装及配置
toc: true
date: 2017-01-05 15:01:54
tags:
categories: note
---
# hadoop搭建

## vmware网络模式
- host-only 虚拟机的网关是物理机
- bridge 所有虚拟机的网关是物理机的网关，虚拟机是与物理机的地位相等
- nat 虚拟出一个网卡，所有的虚拟机都连接在该网卡上面

## 配置linux
1. 编辑sudo
vim /etc/sudoers
2. 改ip
ifconfig 查看网络信息
vim /etc/sysconfig/network-scripts/eth0 更改ip等
curl ifconfig.me 查看公网地址
3. 改启动模式
vim /etc/inittab
4. 配置域名
vim /etc/sysconfig/network 改主机名
scp 源 目的， 远程复制 -r表示强制执行，尤其是复制文件夹的时候
```
scp -r /app/hadoop-2.7.3/etc/ gxq:/app/hadoop-2.7.3
```
5. 关闭防火墙
service iptables stop 关闭防火墙
service iptables status 检查防火墙状态
chkconfig iptables off 关闭防火墙开机启动

## 安装jdk
tar -zxvf jdk-7u55-linux-i586.tar.gz -C /home/hadoop/app C是指定目录
vim /etc/profile
export JAVA_HOME=/home/hadoop/app/jdk-7u_65-i585
export PATH=$PATH:$JAVA_HOME/bin
source /etc/profile


## 安装及配置hadoop
- vim hadoop-env.sh
JAVA_HOME = /home/hadoop/app/jdk_7u65

- vim core-site.xml
最好写主机名，就是host中配置的
```
<configuration>
	<property>
		<name>fs.defaultFS</name>
		<value>hdfs://localhost:9000</value>
		<value>hdfs://115.159.65.146:9000</value>
	</property>
	<property>
		<name>hadoop.tmp.dir</name>
		<value>/home/hadoop/app/hadoop-2.4.1/hdpdata</value>
	</property>
</configuration>
```

- vim mapred-site.xml
 mv mapred-site.xml.template mapred-site.xml 改名
```
<configuration>
	<property>
		<name>mapreduce.framework.name</name>
		<value>yarn</value>
	</property>
</configuration>
```

- vim yarn-site.xml
```
<configuration>
	<property>
		<name>yarn.resourcemanager.hostname</name>
		<value>115.159.65.146</value>
		<!--namenode <value>localhost</value>-->
	</property>

	<property>
		<name>yarn.nodemanager.aux-services</name>
		<value>mapreduce_shuffle</value>
	</property>
</configuration>

```

- vim hdfs-site.xml
这里表示一共有2个副本
```
<configuration>
	<property>
		<name>dfs.replication</name>
		<value>2</value>
	</property>
</configuration>

```
-  vim slaves
文件 slaves，将作为 DataNode 的主机名写入该文件，每行一个，默认为 localhost，配置可以保留 localhost，也可以删掉，让该节点仅作为 NameNode 使用。

```
localhost
hadoop-node01 #已经在hosts中设置好

```

- vim /etc/profile
```
export JAVA_HOME=/app/jdk1.8.0_101
export HADOOP_HOME=/app/hadoop-2.7.3
export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```



## 集群内部的SSH免密登陆
```
ssh-keygen
ssh-copy-id   hdp-node01 //hdp-node01已经在/etc/hosts中配置过
ssh-copy-id   hdp-node02
ssh-copy-id   hdp-node03
```



## 启动hadoop

- 格式化namenode
```
hadoop namenode -format
```

在sbin目录下面有很多自动脚本
```
-rwxr-xr-x 1 root root 2752 Aug 18 09:49 distribute-exclude.sh
-rwxr-xr-x 1 root root 6452 Aug 18 09:49 hadoop-daemon.sh
-rwxr-xr-x 1 root root 1360 Aug 18 09:49 hadoop-daemons.sh
-rwxr-xr-x 1 root root 1597 Aug 18 09:49 hdfs-config.cmd
-rwxr-xr-x 1 root root 1427 Aug 18 09:49 hdfs-config.sh
-rwxr-xr-x 1 root root 2291 Aug 18 09:49 httpfs.sh
-rwxr-xr-x 1 root root 3128 Aug 18 09:49 kms.sh
-rwxr-xr-x 1 root root 4080 Aug 18 09:49 mr-jobhistory-daemon.sh
-rwxr-xr-x 1 root root 1648 Aug 18 09:49 refresh-namenodes.sh
-rwxr-xr-x 1 root root 2145 Aug 18 09:49 slaves.sh
-rwxr-xr-x 1 root root 1727 Aug 18 09:49 start-all.cmd
-rwxr-xr-x 1 root root 1471 Aug 18 09:49 start-all.sh
-rwxr-xr-x 1 root root 1128 Aug 18 09:49 start-balancer.sh
-rwxr-xr-x 1 root root 1360 Aug 18 09:49 start-dfs.cmd
-rwxr-xr-x 1 root root 3734 Aug 18 09:49 start-dfs.sh
-rwxr-xr-x 1 root root 1357 Aug 18 09:49 start-secure-dns.sh
-rwxr-xr-x 1 root root 1524 Aug 18 09:49 start-yarn.cmd
-rwxr-xr-x 1 root root 1347 Aug 18 09:49 start-yarn.sh
-rwxr-xr-x 1 root root 1718 Aug 18 09:49 stop-all.cmd
-rwxr-xr-x 1 root root 1462 Aug 18 09:49 stop-all.sh
-rwxr-xr-x 1 root root 1179 Aug 18 09:49 stop-balancer.sh
-rwxr-xr-x 1 root root 1414 Aug 18 09:49 stop-dfs.cmd
-rwxr-xr-x 1 root root 3206 Aug 18 09:49 stop-dfs.sh
-rwxr-xr-x 1 root root 1340 Aug 18 09:49 stop-secure-dns.sh
-rwxr-xr-x 1 root root 1595 Aug 18 09:49 stop-yarn.cmd
-rwxr-xr-x 1 root root 1340 Aug 18 09:49 stop-yarn.sh
-rwxr-xr-x 1 root root 4295 Aug 18 09:49 yarn-daemon.sh
-rwxr-xr-x 1 root root 1353 Aug 18 09:49 yarn-daemons.sh
```

如果单个启动，可以使用
```
hadoop-daemon.sh start -namenode
hadoop-daemon.sh start -datanode
hadoop-daemon.sh start -secondarynamenode
```
停止时改为stop，yarn同理

如果想一键启动或停止
```
start-dfs.sh
stop-dfs.sh
start-yarn.sh
stop-yarn.sh

//启动hsfs和yarn
start-all.sh
stop-all.sh
```

- jps 仅查找当前用户的Java进程，而不是当前系统中的所有进程
```
netstat -nltp
	-a (all)显示所有选项，默认不显示LISTEN相关
	-t (tcp)仅显示tcp相关选项
	-u (udp)仅显示udp相关选项
	-n 拒绝显示别名，能显示数字的全部转化成数字。
	-l 仅列出有在 Listen (监听) 的服務状态

	-p 显示建立相关链接的程序名
	-r 显示路由信息，路由表
	-e 显示扩展信息，例如uid等
	-s 按各个协议进行统计
	-c 每隔一个固定时间，执行该netstat命令
```



## 错误
- **host文件**
**在namenode中自己的地址是内网ip
在datanode中自己的地址是公网ip**

- rpc端口随机
导致执行mapreduce时候失败，因为服务器中的安全组没有开放

-  拒绝，可能是因为服务器端口没有打开
[root@dy hadoop-2.7.3]# hdfs dfsadmin -report
Java HotSpot(TM) Client VM warning: You have loaded library /app/hadoop-2.7.3/lib/native/libhadoop.so.1.0.0 which might have disabled stack guard. The VM will try to fix the stack guard now.
It's highly recommended that you fix the library with 'execstack -c <libfile>', or link it with '-z noexecstack'.
16/10/03 00:28:25 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
report: Call From dy/115.159.65.146 to dy:9000 failed on connection exception: java.net.ConnectException: Connection refused; For more details see:  http://wiki.apache.org/hadoop/ConnectionRefused

- 前面出现，可能是版本错误
Java HotSpot(TM) Client VM warning: You have loaded library /app/hadoop-2.7.3/lib/native/libhadoop.so.1.0.0 which might have disabled stack guard. The VM will try to fix the stack guard now.
It's highly recommended that you fix the library with 'execstack -c <libfile>', or link it with '-z noexecstack'.

-
org.apache.hadoop.yarn.exceptions.YarnRuntimeException: java.net.BindException: Problem binding to [dy:8031] java.net.BindException: Cannot assign requested address; For more details see:  http://wiki.apache.org/hadoop/BindException
```<property>
  <name>dfs.namenode.rpc-bind-host</name>
  <value>0.0.0.0</value>
  <description>
    The actual address the RPC server will bind to. If this optional address is
    set, it overrides only the hostname portion of dfs.namenode.rpc-address.
    It can also be specified per name node or name service for HA/Federation.
    This is useful for making the name node listen on all interfaces by
    setting it to 0.0.0.0.
  </description>
</property>

<property>
  <name>dfs.namenode.servicerpc-bind-host</name>
  <value>0.0.0.0</value>
  <description>
    The actual address the service RPC server will bind to. If this optional address is
    set, it overrides only the hostname portion of dfs.namenode.servicerpc-address.
    It can also be specified per name node or name service for HA/Federation.
    This is useful for making the name node listen on all interfaces by
    setting it to 0.0.0.0.
  </description>
</property>

<property>
  <name>dfs.namenode.http-bind-host</name>
  <value>0.0.0.0</value>
  <description>
    The actual adress the HTTP server will bind to. If this optional address
    is set, it overrides only the hostname portion of dfs.namenode.http-address.
    It can also be specified per name node or name service for HA/Federation.
    This is useful for making the name node HTTP server listen on all
    interfaces by setting it to 0.0.0.0.
  </description>
</property>

<property>
  <name>dfs.namenode.https-bind-host</name>
  <value>0.0.0.0</value>
  <description>
    The actual adress the HTTPS server will bind to. If this optional address
    is set, it overrides only the hostname portion of dfs.namenode.https-address.
    It can also be specified per name node or name service for HA/Federation.
    This is useful for making the name node HTTPS server listen on all
    interfaces by setting it to 0.0.0.0.
  </description>
</property>
```
同时，要将namenode机器上面的网址全部改为localhost(core-site和yarn-site)，datanode上面的还是写公网ip
