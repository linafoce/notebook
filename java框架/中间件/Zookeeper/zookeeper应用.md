# Zookeeper的基本使用

## Zookeeper的作用



Zookeeper是分布式系统的**协调者**。协调者的作用有点像中介：负责协调房东和租客。



对于分布式系统，要协调什么？分布式系统是由N台服务器，N台服务器之间通过网络进行通信，而网络则是不稳定的，而且会有延迟。



所以，如果A服务器上的数据发生变化，B服务器该如何知道？甚至A服务器如果挂掉了，B服务器该如何知道？



这一切，单单靠A,B两台服务器是无法保证的，需要一个中间人，Zookeeper充当的就是这个角色。



## Zookeeper的安装和使用



### 安装

Zookeeper的官网：https://zookeeper.apache.org/

Zookeeper 3.6.2版本：https://zookeeper.apache.org/doc/r3.6.2/index.html

Zookeeper 所有版本的下载地址：https://zookeeper.apache.org/releases.html

Zookeeper 3.6.2版本的下载地址：https://www.apache.org/dyn/closer.lua/zookeeper/zookeeper-3.6.2/apache-zookeeper-3.6.2-bin.tar.gz



#### 下载

![image.png](https://cdn.nlark.com/yuque/0/2020/png/365147/1609230000143-741db004-b310-4e01-9e50-df6bfde9f498.png?x-oss-process=image%2Fresize%2Cw_1500)

#### 解压

```
tar -zxvf apache-zookeeper-3.6.2-bin.tar.gz
```

![image.png](https://cdn.nlark.com/yuque/0/2020/png/365147/1609230196426-b4552766-655c-44a7-a89f-5ee816d18a50.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_6bKB54-t5a2m6Zmi5Ye65ZOB%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10%2Fresize%2Cw_1500)



可以将解压后的文件夹移动到你想要移动的位置

![image.png](https://cdn.nlark.com/yuque/0/2020/png/365147/1609230314644-c3b07a60-7dce-4ee2-9970-3cbaa85168b3.png)



最后，我的zookeeper的安装路径为：/root/software/apache-zookeeper-3.6.2-bin

![image.png](https://cdn.nlark.com/yuque/0/2020/png/365147/1609231012635-ef6bbc80-a840-4682-8069-4780e41783f4.png)



在安装路径下的bin目录下，就是zookeeper提供的执行命令：

![image.png](https://cdn.nlark.com/yuque/0/2020/png/365147/1609231054561-9f228240-842f-4dd0-b7ff-fd9bcf38db0d.png)



常用的是：

1. zkCli.sh：linux客户端
2. zkCli.cmd：windows客户端
3. zkServer.sh：linux服务端
4. zkServer.cmd：windows服务端



建议把bin目录添加到操作系统的环境变量中去，编辑 ~/.bash_profile

```
vi ~/.bash_profile 
```

![image.png](https://cdn.nlark.com/yuque/0/2020/png/365147/1609231346616-30babbd3-ae46-4137-a3de-ba00423591b0.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_6bKB54-t5a2m6Zmi5Ye65ZOB%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10%2Fresize%2Cw_1500)



这样可以比较方便的来使用zookeeper



### 启动

#### 启动服务端

在启动服务端之前，得准备一份zookeeper要用的配置文件，在zookeeper的安装路径下有conf目录：



![image.png](https://cdn.nlark.com/yuque/0/2020/png/365147/1609231888253-2ed35919-8267-4557-9936-08481d6034d6.png?x-oss-process=image%2Fresize%2Cw_1500)

我们可以直接把这个配置文件复制一份为zoo.cfg，zoo.cfg才是zookeeper真正会去使用的配置文件

![image.png](https://cdn.nlark.com/yuque/0/2020/png/365147/1609231964983-c260c633-496f-4352-9f3f-7fa9e3d5c2c4.png?x-oss-process=image%2Fresize%2Cw_1500)



有了配置文件后可以启动zookeeper（以下简称zk）服务端了





注意：以下都是基于linux的，如果你想在windows上使用，缓存对应的**.cmd脚本



![image.png](https://cdn.nlark.com/yuque/0/2020/png/365147/1609232062720-59fd8b54-1c43-4b14-bc86-ec326dbbffd6.png?x-oss-process=image%2Fresize%2Cw_1500)

控制台显示STARTED，但是不一定真的成功了，具体还要看对应的启动日志，默认日志目录和日志文件为：



![image.png](https://cdn.nlark.com/yuque/0/2020/png/365147/1609233259816-6b58d30c-182d-423a-8ef7-995573386870.png?x-oss-process=image%2Fresize%2Cw_1500)



直接

```
cat zookeeper-root-server-localhost.localdomain.out 
```

就能查看日志。



Zookeeper也是用Java实现的，所以如果没有在日志中看到异常信息，那基本上就启动成功了。



**请记住这个日志文件，这是排查Zookeeper问题的关键。**



当然，还有一种方式就是可以利用jps命令查看运行中的java进程，

![image.png](https://cdn.nlark.com/yuque/0/2020/png/365147/1609233774227-e2bfedf5-d2e9-4875-bd9b-4ab7a0156814.png?x-oss-process=image%2Fresize%2Cw_1500)

QuorumPeerMain其实是Zookeeper服务端的启动类，16899表示进程号，只要有这个信息，基本Zookeeper服务端就是正常运行中的。





#### 启动客户端



直接运行`zkCli.sh`脚本就能启动客户端



![image.png](https://cdn.nlark.com/yuque/0/2020/png/365147/1609240577645-606d3424-495f-41ac-a174-b7d9d46057b0.png?x-oss-process=image%2Fresize%2Cw_1500)

![image.png](https://cdn.nlark.com/yuque/0/2020/png/365147/1609240597982-c960453b-2a47-48c2-8d91-939ca1f43ace.png?x-oss-process=image%2Fresize%2Cw_1500)



一旦连接成功，此处就会显示CONNECTED，如果是连接中，就会显示CONNECTING（表示服务端没启动成功，客户端连接不上，一直在重试）





如果你想客户端连指定的Zookeeper服务端，也是可以的通过-server参数可以指定hostname:port

![image.png](https://cdn.nlark.com/yuque/0/2020/png/365147/1609240794738-1edaa4a1-2cdc-4979-890a-492921d444d5.png?x-oss-process=image%2Fresize%2Cw_1500)



## 节点类型



Zookeeper可以存取数据，Mysql存的数据就做“行”，Redis中存的数据叫“kv对”，Zookeeper中存的数据叫“znode节点”。一个znode有节点名字、节点内容、子节点。我们可以自由的增加、删除znode，在一个znode下增加、删除子znode。



有四种类型的znode：



### PERSISTENT（持久化节点）

客户端与zookeeper断开连接后，该节点依旧存在，只要不手动删除该节点，他将永远存在



### PERSISTENT_SEQUENTIAL（持久化顺序节点）

客户端与zookeeper断开连接后，该节点依旧存在，只是Zookeeper给该节点名称进行顺序编号



### EPHEMERAL（临时节点）

客户端与zookeeper断开连接后，该节点被删除。



1. 临时节点下不能拥有子节点
2. 其他客户端也能看到临时节点





### EPHEMERAL_SEQUENTIAL（临时顺序节点）

客户端与zookeeper断开连接后，该节点被删除，只是Zookeeper给该节点名称进行顺序编号



### Container（容器节点）

3.5.3 版本新增，如果Container节点下面没有子节点，则Container节点在未来会被Zookeeper自动清除,定时任务默认60s 检查一次 



### PERSISTENT_WITH_TTL（持久化TTL节点）

3.5.3 版本新增，默认禁用，只能通过系统配置 zookeeper.extendedTypesEnabled=true 开启，拥有一个过期时间



### PERSISTENT_SEQUENTIAL_WITH_TTL（持久化TTL顺序节点）

3.5.3 版本新增，默认禁用，只能通过系统配置 zookeeper.extendedTypesEnabled=true 开启，在顺序节点的基础上增加了过期时间概念





## 基本的CRUD



### 创建节点（znode）



语法：

```
create /path data
```



示例：

```
create /FirstNode first
```



输出：

```
[zk: localhost:2181(CONNECTED) 3] create /FirstNode first
Created /FirstNode
```



要创建顺序节点，请添加flag：-s，如下所示。

语法：

```
create -s /path data
```



示例：

```
create -s /FirstNode second
```



输出：

```
[zk: localhost:2181(CONNECTED) 4] create -s /FirstNode second
Created /FirstNode0000000018
```



要创建临时节点，请添加flag：-e ，如下所示。



语法：

```
create -e /path data
```



示例：

```
create -e /FirstNode-ephemeral ephemeral
```



输出：

```
[zk: localhost:2181(CONNECTED) 6] create -e /FirstNode-ephemeral ephemeral
Created /FirstNode-ephemeral
```



记住当客户端断开连接时，临时节点将被删除。你可以通过退出ZooKeeper CLI，然后重新打开CLI来尝试。



### 创建容器节点

```
create -c /path data
```



只有添加过子节点，容器节点的特性才会生效，容器节点的特性是：节点的最后一个子节点被删除过，该节点会自动删除（可能延迟一段时间）



### 创建TTL节点

```
create -t 3000 /path data
```



TTL节点创建后，如果3秒内没有数据修改，并且没有子节点，则会自动删除



> 前提是，服务端支持了TTL节点，默认没有开启，通过-Dzookeeper.extendedTypesEnabled=true可以开启





### 获取数据



语法：

```
get /path
```



示例：

```
get /FirstNode
```



输出：

```
[zk: localhost:2181(CONNECTED) 7] get /FirstNode
first
cZxid = 0xa2
ctime = Wed Dec 12 13:29:14 CST 2018
mZxid = 0xa2
mtime = Wed Dec 12 13:29:14 CST 2018
pZxid = 0xa2
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 5
numChildren = 0
```



要访问顺序节点，必须输入znode的完整路径。



示例：

```
get /FirstNode0000000018
```



输出：

```
[zk: localhost:2181(CONNECTED) 9] get /FirstNode0000000018
second
cZxid = 0xa3
ctime = Wed Dec 12 13:30:44 CST 2018
mZxid = 0xa3
mtime = Wed Dec 12 13:30:44 CST 2018
pZxid = 0xa3
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 6
numChildren = 0
```



### 设置数据



语法：

```
set /path /data
```



示例：

```
set /FirstNode first_update
```



输出：

```
[zk: localhost:2181(CONNECTED) 12] set /FirstNode first_update

WATCHER::

WatchedEvent state:SyncConnected type:NodeDataChanged path:/FirstNode
cZxid = 0xa2
ctime = Wed Dec 12 13:29:14 CST 2018
mZxid = 0xa6
mtime = Wed Dec 12 13:51:39 CST 2018
pZxid = 0xa2
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 12
numChildren = 0
```



### 创建子节点



语法：

```
create /parent/path/subnode/path data
```



示例：

```
create /FirstNode/Child firstchildren
```



输出：

```
[zk: localhost:2181(CONNECTED) 13] create /FirstNode/Child firstchildren
Created /FirstNode/Child
[zk: localhost:2181(CONNECTED) 14] create /FirstNode/Child2 secondchildren
Created /FirstNode/Child2
```



### 列出子节点



语法：

```
ls /path
```



实例：

```
ls /FirstNode
```



输出：

```
[zk: localhost:2181(CONNECTED) 15] ls /FirstNode
[Child2, Child]
```



### 检查状态



语法：

```
stat /path
```



示例：

```
stat /FirstNode
```



输出：

```
[zk: localhost:2181(CONNECTED) 16] stat /FirstNode
cZxid = 0xa2
ctime = Wed Dec 12 13:29:14 CST 2018
mZxid = 0xa6
mtime = Wed Dec 12 13:51:39 CST 2018
pZxid = 0xa8
cversion = 2
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 12
numChildren = 2
```



### 移除Znode



语法：

```
delete /path
```



示例：

```
delete /FirstNode
```



输出：

```
[zk: localhost:2181(CONNECTED) 17] rmr /FirstNode 
[zk: localhost:2181(CONNECTED) 18] get /FirstNode
Node does not exist: /FirstNode
```



删除(delete /path)命令类似于 remove 命令，但是只适用于没有子节点的znode。



## 权限管理ACL



zk做为分布式架构中的重要中间件，通常会在上面以节点的方式存储一些关键信息，默认情况下，所有应用都可以读写任何节点，在复杂的应用中，这不太安全，ZK通过ACL机制来解决访问权限问题。



- ZooKeeper的权限控制是基于每个znode节点的，需要对每个节点设置权限
- 每个znode支持设置多种权限控制方案和多个权限
- 子节点不会继承父节点的权限，客户端无权访问某节点，但可能可以访问它的子节点



ACL 权限控制，使用：schema:id:permission 来标识，主要涵盖 3 个方面：



- 权限模式（Schema）：鉴权的策略
- 授权对象（ID）
- 权限（Permission）



### schema



- world：只有一个用户：anyone，代表所有人（默认）
- ip：使用IP地址认证
- auth：使用已添加认证的用户认证
- digest：使用“用户名:密码”方式认证



### id



授权对象ID是指，权限赋予的用户或者一个实体，例如：IP 地址或者机器。授权模式 schema 与 授权对象 ID 之间关系：



- world：只有一个id，即anyone
- ip: 通常是一个ip地址或地址段，比如192.168.0.110或192.168.0.1/24
- auth：用户名
- digest：自定义：通常是"username:BASE64(SHA-1(username:password))"



### 权限



- CREATE, 简写为c，可以创建子节点
- DELETE，简写为d，可以删除子节点（仅下一级节点），注意不是本节点
- READ，简写为r，可以读取节点数据及显示子节点列表
- WRITE，简写为w，可设置节点数据
- ADMIN，简写为a，可以设置节点访问控制列表



#### 查看ACL



```
getAcl /parent
```



返回



```
[zk: localhost:2181(CONNECTED) 122] getAcl /parent
'world,'anyone
: cdrwa
```



默认创建的节点的权限是最开放的，所有都可以增删查改管理。



#### 设置ACL



设置节点对所有人都有删（d）和管理权限（a）



```
setAcl /parent world:anyone:da
```



所以去读取数据的时候会提示



```
[zk: localhost:2181(CONNECTED) 124] get /parent
Authentication is not valid : /parent
```



先添加用户：



```
addauth digest zhangsan:12345
```



再设置权限，这个节点只有zhangsan这个用户拥有所有权限



```
setAcl /parent auth:zhangsan:12345:rdwca
```



### 超级管理员

超级管理员的用户名为super，密码自定义比如：admin



1. 首先调用DigestAuthenticationProvider.generateDigest("super:admin")获取签名，比如结果为：super:xQJmxLMiHGwaqBvst5y6rkB6HQs=
2. 在启动Zookeeper服务端（zkServer.sh脚本中）时加入-Dzookeeper.DigestAuthenticationProvider.superDigest=super:xQJmxLMiHGwaqBvst5y6rkB6HQs=
3. 启动zookeeper并使用客户端进行连接
4. 如果遇到没有操作权限的节点，这时可以addauth digest super:admin来开启管理员，即有所有权限





## 配置文件详解

- tickTime：Zookeeper 服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每个 tickTime 时间就会发送一个心跳。tickTime以毫秒为单位。该参数用来定义心跳的间隔时间，zookeeper的客户端和服务端之间也有和web开发里类似的session的概念，而zookeeper里最小的session过期时间就是tickTime的两倍。
- initLimit：Follower在启动过程中，会从Leader同步所有最新数据，然后确定自己能够对外服务的起始状态。Leader允许Follower在 initLimit 时间内完成这个工作。通常情况下，我们不用太在意这个参数的设置。如果ZK集群的数据量确实很大了，Follower在启动的时候，从Leader上同步数据的时间也会相应变长，因此在这种情况下，有必要适当调大这个参数了。默认为10。
- syncLimit：在运行过程中，Leader负责与ZK集群中所有机器进行通信，例如通过一些心跳检测机制，来检测机器的存活状态。如果Leader发出心跳包在syncLimit之后，还没有从Follower那里收到响应，那么就认为这个Follower已经不在线了。
- dataDir：存储快照文件snapshot的目录。默认情况下，事务日志也会存储在这里。建议同时配置参数dataLogDir, 事务日志的写性能直接影响zk性能。
- clientPort：客户端连接服务器的端口
- maxClientCnxns：Zookeeper服务端同时支持在线的客户端数量限制
- autopurge.snapRetainCount：Zookeeper服务端自动清除多余的日志和快照文件时至少保留的快照数量
- autopurge.purgeInterval：Zookeeper服务端自动清除多余的日志和快照文件的周期