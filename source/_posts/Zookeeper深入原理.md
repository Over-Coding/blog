---
title: Zookeeper深入原理
date: 2020-04-18 14:50:30
categories:
	- Zookeeper
tags: 
	- Zookeeper
	- 深入原理
toc: true
typora-root-url: Zookeeper深入原理
---

> 本篇文章摘自《从Paxos到Zookeeper分布式一致性原理与实践》

### 1、系统模型

#### 1.1、数据模型

Zookeeper 的视图结构是一个树形结构，树上的每个节点称之为数据节点（即 ZNode），每个ZNode 上都可以保存数据，同时还可以挂载子节点。并且Zookeeper的根节点为 "/"。

![](zookeeper数据结构.png)



#### 1.2、节点类型

在 Zookeeper 中，每个数据节点都是有生命周期的，其生命周期的长短取决于数据节点的节点类型。在 Zookeeper 中有如下几类节点：

| 节点类型                              | 说明                                                         |
| ------------------------------------- | ------------------------------------------------------------ |
| 持久节点（PERSISTENT）                | 指该数据节点被创建后，就会一直存在于 Zookeeper 服务器上，直到有删除操作来主动清除这个节点。 |
| 持久顺序节点（PERSISTENT_SEQUENTIAL） | 基本特性和持久节点是一致的，额外的特性表现在顺序性上，在 Zookeeper 中，每个父节点都会为它的第一级子节点维护一份顺序，用于记录下每个子节点创建的先后顺序。基于这个顺序特性，在创建子节点的时候，可以设置这个标记，那么在创建节点过程中，Zookeeper 会自动为给定节点名加上一个数字后缀，作为一个新的、完整的节点名。**另外需要注意的是，这个数字后缀的上限是整型的最大值。** |
| 临时节点（EPHEMERAL）                 | 临时节点的生命周期和客户端的会话绑定在一起，如果客户端会话失效，那么这个节点就会被自动清理掉。**另外，Zookeeper 规定了不能基于临时节点来创建子节点，即临时节点只能作为叶子节点。** |
| 临时顺序节点（EPHEMERAL_SEQUENTIAL）  | 基本特性和临时节点一致，只是添加了顺序的特性。               |



#### 1.3、状态信息

每个数据节点中除了存储了数据内容之外，还存储了数据节点本身的一些状态信息（State）。

| 状态属性       | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| cZxid          | 即 Create ZXID，表示该数据节点被创建时的事务ID。             |
| ctime          | 即 Create Time，表示该数据节点被创建的时间。                 |
| mZxid          | 即 Modified ZXID，表示该节点最后一次被更新时的事务ID。       |
| mtime          | 即 Modified Time，表示该数据节点最后一次被更新的时间。       |
| pZxid          | 表示该节点的子节点列表最后一次被修改时的事务ID。注意，只有子节点列表变更了才会变更 pZxid，子节点内容变更不会影响pZxid。 |
| cversion       | 表示子节点的版本号。                                         |
| dataVersion    | 表示数据节点的版本号。                                       |
| aclVersion     | 表示节点的 ACL 版本号。                                      |
| ephemeralOwner | 创建该临时节点的会话的sessionID。如果该节点是持久节点，那么这个属性值为0。 |
| dataLength     | 表示数据内容的长度。                                         |
| numChildren    | 表示当前节点的子节点个数。                                   |



#### 1.4、ZXID

在Zookeeper 中，事务是指能够改变 Zookeeper 服务器状态的操作，我们也称之为事务操作或更新操作，一般包括数据节点创建与删除、数据节点内容更新和客户端会话创建与失效等操作。对于每一个事务请求，Zookeeper 都会为其分配一个全局唯一的事务ID，用 ZXID 来表示，通常是一个 64 位的数字。每一个 ZXID 对应一次更新操作，从这些 ZXID 中可以间接地识别出 Zookeeper 处理这些更新操作请求的全局顺序。

ZXID 是一个 64 位的数字，其中低 32 位可以看作是一个简单的单调递增的计数器，针对客户端的每一个事务请求，Leader 服务器在产生一个新的事务 Proposal 的时候，都会对该计数器进行加 1 操作；而高 32 位则代表了 Leader 周期 epoch 的编号，每当选举产生一个新的 Leader 服务器，就会从这个 Leader 服务器上取出其本地日志中最大事务 Proposal 的 ZXID，并从该 ZXID 中解析出对应的 epoch 值，然后再对其进行加 1 操作，之后就会以此编号作为新的 epoch，并将低 32 位置 0 来开始生成新的 ZXID。



#### 1.5、版本

 Zookeeper 中为数据节点引入了版本的概念，每个数据节点都具有三种类型的版本信息（在上面的状态信息中已经介绍了三种版本信息代表的意思），对数据节点的任何更新操作都会引起版本号的变化。其中我们以 dataVersion 为例来说明。在一个数据节点被创建完毕之后，节点的dataVersion 值是 0，表示的含义是 ”当前节点自从创建之后，被更新过 0 次“。如果现在对该节点的数据内容进行更新操作，那么随后，dataVersion 的值就会变成 1。即表示的是对数据节点的数据内容的变更次数。

版本的作用是用来实现乐观锁机制中的 “写入校验” 的。例如，当要修改数据节点的数据内容时，带上版本号，如果数据节点的版本号与传入的版本号相等，就进行修改，否则修改失败。



#### 1.6、Watcher

##### 1.6.1、概述

Zookeeper 提供了分布式数据的发布/订阅功能。一个典型的发布/订阅模型系统定义了一种一对多的订阅关系，能够让多个订阅者同时监听某一个主题对象，当这个主题对象自身状态变化时，会通知所有订阅者，使它们能够做出相应的处理。在 Zookeeper 中，引入了 Watcher 机制来实现这种分布式的通知功能。Zookeeper 允许客户端向服务端注册一个 Watcher 监听，当服务端的一些指定事件触发了这个 Watcher，那么就会向指定客户端发送一个事件通知来实现分布式的通知功能。

![](Watcher.png)

从上图可以看出 Zookeeper 的 Watcher 机制主要包括客户端线程、客户端WatchMananger 和 Zookeeper 服务器三部分。在具体工作流程上，简单地讲，客户端在向 Zookeeper 服务器注册 Watcher 的同时，会将 Watcher 对象存储在客户端的 WatchMananger 中。当 Zookeeper 服务器端触发 Watcher 事件后，会向客户端发送通知，客户端线程从 WatchManager 中取出对应的 Watcher 对象来执行回调逻辑。



##### 1.6.2、Watcher特性

* **一次性：**表示无论是服务端还是客户端，一旦一个 Watcher 被触发，Zookeeper 都会将其从相应的存储中移除。因此，开发人员在 Watcher 的使用上要记住的一点是需要反复注册。
* **客户端串行执行：**客户端 Watcher 回调的过程是一个串行同步的过程，这为我们保证了顺序，同时，需要开发人员注意的一点是，千万不要因为一个 Watcher 的处理逻辑影响了整个客户端的 Watcher 回调。
* **轻量：**WatchedEvent 是 Zookeeper 整个 Watcher 通知机制的最小通知单元，这个数据结构中只包含三部分内容：通知状态、事件类型和节点路径。也就是说，Watcher通知非常简单，只会告诉客户端发生了事件，而不会说明事件的具体内容。



##### 1.6.3、watcher接口设计

Watcher是一个接口，任何实现了Watcher接口的类就是一个新的Watcher。Watcher内部包含了两个枚举类：KeeperState、EventType

- **Watcher通知状态(KeeperState)**

  KeeperState是客户端与服务端连接状态发生变化时对应的通知类型。路径为org.apache.zookeeper.Watcher.Event.KeeperState，是一个枚举类，其枚举属性如下：

| 枚举属性      | 说明                     |
| ------------- | ------------------------ |
| SyncConnected | 客户端与服务器正常连接时 |
| Disconnected  | 客户端与服务器断开连接时 |
| Expired       | 会话session失效时        |
| AuthFailed    | 身份认证失败时           |

- **Watcher事件类型(EventType)**

  EventType是数据节点(znode)发生变化时对应的通知类型。EventType变化时KeeperState永远处于SyncConnected通知状态下；当KeeperState发生变化时，EventType永远为None。其路径为org.apache.zookeeper.Watcher.Event.EventType，是一个枚举类，枚举属性如下：

| 枚举属性            | 说明                                                      |
| ------------------- | --------------------------------------------------------- |
| None                | 无                                                        |
| NodeCreated         | Watcher监听的数据节点被创建时                             |
| NodeDeleted         | Watcher监听的数据节点被删除时                             |
| NodeDataChanged     | Watcher监听的数据节点内容发生变更时(无论内容数据是否变化) |
| NodeChildrenChanged | Watcher监听的数据节点的子节点列表发生变更时               |

**注**：客户端接收到的相关事件通知中只包含状态及类型等信息，不包括节点变化前后的具体内容，变化前的数据需业务自身存储，变化后的数据需调用get等方法重新获取；



##### 1.6.4、捕获相应的事件

上面讲到zookeeper客户端连接的状态和zookeeper对znode节点监听的事件类型，下面我们来讲解如何建立zookeeper的watcher监听。在zookeeper中采用zk.getChildren(path, watch)、zk.exists(path, watch)、zk.getData(path, watcher, stat)这样的方式为某个znode注册监听。

下表以node-x节点为例，说明调用的注册方法和可监听事件间的关系：

| 注册方式                          | Created | ChildrenChanged | Changed | Deleted |
| :-------------------------------- | :------ | :-------------- | :------ | :------ |
| zk.exists(“/node-x”,watcher)      | 可监控  |                 | 可监控  | 可监控  |
| zk.getData(“/node-x”,watcher)     |         |                 | 可监控  | 可监控  |
| zk.getChildren(“/node-x”,watcher) |         | 可监控          |         | 可监控  |



#### 1.7、ACL

Zookeeper 中提供了一套完善的 ACL（Access Control List）权限控制机制来保障数据的安全。

##### 1.7.1、概述

ACL 由三部分组成，分别是：权限模式（Scheme）、授权对象（ID）和权限（Permission），通常使用“scheme: ​id:permission”来标识一个有效的ACL 信息。下面分别介绍：

1. **权限模式（Scheme）**

   | 方案   | 说明                                                    |
   | ------ | ------------------------------------------------------- |
   | world  | 只有一个用户：anyone，代表登录 Zookeeper 所有人（默认） |
   | ip     | 对客户端使用IP地址认证。                                |
   | auth   | 使用已添加认证的用户认证。                              |
   | digest | 使用“用户名:密码”方式认证。                             |

   

2. **授权对象（ID）**

   授权对象ID是指，权限赋予的实体，例如：IP 地址或用户。

   

3. **权限（Permission）**

   | 权限   | ACL简写 | 描述                             |
   | ------ | ------- | -------------------------------- |
   | create | c       | 可以创建子节点。                 |
   | delete | d       | 可以删除子节点（仅下一级节点）。 |
   | read   | r       | 可以读取节点数据或子节点列表。   |
   | write  | w       | 可以对节点进行更新操作。         |
   | admin  | a       | 可以设置节点访问控制列表权限。   |

   

##### 1.7.2、特性

* zooKeeper的权限控制是基于每个znode节点的，需要对每个节点设置权限。
* 每个znode支持设置多种权限控制方案和多个权限。
* 子节点不会继承父节点的权限，客户端无权访问某节点，但可能可以访问它的子节点。



##### 1.7.3、案例

* world授权模式

  命令

  ```shell
  setAcl <path> world:anyone:<acl>
  ```

  案例

  ```shell
  [zk: localhost:2181(CONNECTED) 0] create /node1 "node1"
  Created /node1
  [zk: localhost:2181(CONNECTED) 1] getAcl /node1
  'world,'anyone
  : cdrwa
  [zk: localhost:2181(CONNECTED) 2] setAcl /node1 world:anyone:crwa
  cZxid = 0x100000004
  ctime = Fri May 29 14:31:54 CST 2020
  mZxid = 0x100000004
  mtime = Fri May 29 14:31:54 CST 2020
  pZxid = 0x100000004
  cversion = 0
  dataVersion = 0
  aclVersion = 1
  ephemeralOwner = 0x0
  dataLength = 5
  numChildren = 0
  ```

* IP授权模式

  命令

  ```shell
  setAcl <path> ip:<ip>:<acl>
  ```

  案例

  注意：远程登录zookeeper命令:./zkCli.sh -server ip

  ```
  [zk: localhost:2181(CONNECTED) 18] create /node2 "node2"
  Created /node2
  
  [zk: localhost:2181(CONNECTED) 23] setAcl /node2 ip:192.168.150.101:cdrwa
  cZxid = 0xe
  ctime = Fri Dec 13 22:30:29 CST 2019
  mZxid = 0x10
  mtime = Fri Dec 13 22:33:36 CST 2019
  pZxid = 0xe
  cversion = 0
  dataVersion = 2
  aclVersion = 1
  ephemeralOwner = 0x0
  dataLength = 20
  numChildren = 0
  
  [zk: localhost:2181(CONNECTED) 25] getAcl /node2
  'ip,'192.168.150.101
  : cdrwa
  
  #使用IP非 192.168.150.101 的机器
  [zk: localhost:2181(CONNECTED) 0] get /node2
  Authentication is not valid : /node2 #没有权限
  ```

* Auth授权模式

  命令

  ```shell
  addauth digest <user>:<password> #添加认证用户
  setAcl <path> auth:<user>:<acl>
  ```

  案例

  ```shell
  [zk: localhost:2181(CONNECTED) 6] create /node3 "node3"
  Created /node3
  
  #添加认证用户
  [zk: localhost:2181(CONNECTED) 7] addauth digest ld:123456
  
  [zk: localhost:2181(CONNECTED) 8] setAcl /node3 auth:ld:cdrwa
  cZxid = 0x10000000c
  ctime = Fri May 29 14:47:13 CST 2020
  mZxid = 0x10000000c
  mtime = Fri May 29 14:47:13 CST 2020
  pZxid = 0x10000000c
  cversion = 0
  dataVersion = 0
  aclVersion = 1
  ephemeralOwner = 0x0
  dataLength = 5
  numChildren = 0
  
  [zk: localhost:2181(CONNECTED) 9] getAcl /node3
  'digest,'ld:kesl2p6Yx58a+/mP+TKSFZkzkZ0=
  : cdrwa
  
  #添加认证用户后可以访问
  [zk: localhost:2181(CONNECTED) 10] get /node3
  node3
  cZxid = 0x10000000c
  ctime = Fri May 29 14:47:13 CST 2020
  mZxid = 0x10000000c
  mtime = Fri May 29 14:47:13 CST 2020
  pZxid = 0x10000000c
  cversion = 0
  dataVersion = 0
  aclVersion = 1
  ephemeralOwner = 0x0
  dataLength = 5
  numChildren = 0
  ```

* Digest授权模式

  命令

  ```shell
  setAcl <path> digest:<user>:<password>:<acl>
  ```

  这里的密码是经过SHA1及BASE64处理的密文，在SHELL中可以通过以下命令计算：

  ```shell
  echo -n <user>:<password> | openssl dgst -binary -sha1 | openssl base64
  ```

  先来计算一个密文

  ```shell
  echo -n monkey:123456 | openssl dgst -binary -sha1 | openssl base64
  ```

  案例

  ```shell
  [zk: localhost:2181(CONNECTED) 12] create /node4 "node4"
  Created /node4
  
  [zk: localhost:2181(CONNECTED) 13] setAcl /node4 digest:monkey:Rk6u/zJJdOYrTZ6+J0p4/4gTILg=:cdrwa
  cZxid = 0x10000000e
  ctime = Fri May 29 14:52:50 CST 2020
  mZxid = 0x10000000e
  mtime = Fri May 29 14:52:50 CST 2020
  pZxid = 0x10000000e
  cversion = 0
  dataVersion = 0
  aclVersion = 1
  ephemeralOwner = 0x0
  dataLength = 5
  numChildren = 0
  
  #没有权限无法读取
  [zk: localhost:2181(CONNECTED) 14] getAcl /node4
  Authentication is not valid : /node4
  
  #添加认证用户
  [zk: localhost:2181(CONNECTED) 15] addauth digest monkey:123456
  
  [zk: localhost:2181(CONNECTED) 16] getAcl /node4               
  'digest,'monkey:Rk6u/zJJdOYrTZ6+J0p4/4gTILg=
  : cdrwa
  
  [zk: localhost:2181(CONNECTED) 17] get /node4
  node4
  cZxid = 0x10000000e
  ctime = Fri May 29 14:52:50 CST 2020
  mZxid = 0x10000000e
  mtime = Fri May 29 14:52:50 CST 2020
  pZxid = 0x10000000e
  cversion = 0
  dataVersion = 0
  aclVersion = 1
  ephemeralOwner = 0x0
  dataLength = 5
  numChildren = 0
  ```

* 多种模式授权

  同一个节点可以同时使用多种模式授权

  ```shell
  [zk: localhost:2181(CONNECTED) 18] create /node5 "node5"
  Created /node5
  [zk: localhost:2181(CONNECTED) 19] addauth digest ld:123456
  [zk: localhost:2181(CONNECTED) 20] setAcl /node5 ip:192.168.150.101:cdrwa,auth:ld:cdrwa
  cZxid = 0x100000010
  ctime = Fri May 29 14:56:38 CST 2020
  mZxid = 0x100000010
  mtime = Fri May 29 14:56:38 CST 2020
  pZxid = 0x100000010
  cversion = 0
  dataVersion = 0
  aclVersion = 1
  ephemeralOwner = 0x0
  dataLength = 5
  numChildren = 0
  ```



##### 1.7.4、ACL 超级管理员

zookeeper的权限管理模式有一种叫做super，该模式提供一个超管可以方便的访问任何权限的节点

假设这个超管是：super:admin，需要先为超管生成密码的密文

```shell
echo -n super:admin | openssl dgst -binary -sha1 | openssl base64
```

那么打开zookeeper目录下的/bin/zkServer.sh服务器脚本文件，找到如下一行：

```sh
nohup $JAVA "-Dzookeeper.log.dir=${ZOO_LOG_DIR}" "-Dzookeeper.root.logger=${ZOO_LOG4J_PROP}"
```

这就是脚本中启动zookeeper的命令，默认只有以上两个配置项，我们需要加一个超管的配置项

```sh
"-Dzookeeper.DigestAuthenticationProvider.superDigest=super:xQJmxLMiHGwaqBvst5y6rkB6HQs="
```

那么修改以后这条完整命令变成了

```sh
nohup $JAVA "-Dzookeeper.log.dir=${ZOO_LOG_DIR}" "-Dzookeeper.root.logger=${ZOO_LOG4J_PROP}" "-Dzookeeper.DigestAuthenticationProvider.superDigest=super:xQJmxLMiHGwaqBvst5y6rkB6HQs="\
    -cp "$CLASSPATH" $JVMFLAGS $ZOOMAIN "$ZOOCFG" > "$_ZOO_DAEMON_OUT" 2>&1 < /dev/null &
```

之后启动zookeeper,输入如下命令添加权限

```shell
addauth digest super:admin #添加认证用户
```



### 2、Leader 选举

#### 2.1、服务器状态

* looking：寻找leader状态。当服务器处于该状态时，它会认为当前集群中没有Leader，因此需要进入 Leader 选举流程。
* leading：领导者状态。表明当前服务器角色是leader。
* following：跟随者状态。表明当前服务器角色是follower。
* observing：观察者状态。表明当前服务器角色是observer。



#### 2.2、服务器启动时期的 Leader 选举

在服务器集群初始化阶段，我们以 3 台机器组成的服务器集群为例，当有一台服务器server1 启动的时候，它是无法进行 Leader 选举的，当第二台机器 server2 也启动时，此时这两台服务器已经能够进行互相通信，每台机器都试图找到一个 Leader，于是便进入了 Leader 选举流程。

1. 每个server发出一个投票。由于是初始情况，server1和server2都会将自己作为leader服务器来进行投票，每次投票会包含所推举的服务器的myid和zxid，使用(myid, zxid)来表示，此时server1的投票为(1, 0)，server2的投票为(2, 0)，然后各自将这个投票发给集群中其他机器。

2. 集群中的每台服务器接收来自集群中各个服务器的投票。

3. 处理投票。针对每一个投票，服务器都需要将别人的投票和自己的投票进行pk，pk规则如下

   - 优先检查zxid。zxid比较大的服务器优先作为leader。
   - 如果zxid相同，那么就比较myid。myid较大的服务器作为leader服务器。

   ​       对于Server1而言，它的投票是(1, 0)，接收Server2的投票为(2, 0)，首先会比较两者的zxid，均为0，再比较myid，此时server2的myid最大，于是更新自己的投票为(2, 0)，然后重新投票，对于server2而言，其无须更新自己的投票，只是再次向集群中所有机器发出上一次投票信息即可。

4. 统计投票。每次投票后，服务器都会统计投票信息，判断是否已经有过半机器接受到相同的投票信息，对于server1、server2而言，都统计出集群中已经有两台机器接受了(2, 0)的投票信息，此时便认为已经选出了leader。

5. 改变服务器状态。一旦确定了leader，每个服务器就会更新自己的状态，如果是follower，那么就变更为following，如果是leader，就变更为leading。



#### 2.3、服务器运行时期的 Leader 选举

在zookeeper运行期间，leader与非leader服务器各司其职，即便当有非leader服务器宕机或新加入，此时也不会影响leader，但是一旦leader服务器挂了，那么整个集群将暂停对外服务，进入新一轮leader选举，其过程和启动时期的Leader选举过程基本一致。

假设正在运行的有server1、server2、server3三台服务器，当前leader是server2，若某一时刻leader挂了，此时便开始Leader选举。选举过程如下:

1. 变更状态。leader挂后，余下的非 Observer 服务器都会将自己的服务器状态变更为looking，然后开始进入leader选举过程。
2. 每个server会发出一个投票。在运行期间，每个服务器上的zxid可能不同，此时假定server1的zxid为123，server3的zxid为122，在第一轮投票中，server1和server3都会投自己，产生投票(1, 123)，(3, 122)，然后各自将投票发送给集群中所有机器。
3. 接收来自各个服务器的投票。
4. 处理投票。对于投票的处理，和上面提到的服务器启动期间的处理规则是一致的。在这个例子里面，由于 Server1 的 zxid 为 123，Server3 的 zxid 为 122，那么显然，Server1 会成为 Leader。
5. 统计投票。
6. 改变服务器状态。



#### 2.4、Observer 角色及其设置

observer角色特点：

1. 不参与集群的leader选举
2. 不参与集群中写数据时的ack反馈

为了使用observer角色，在任何想变成observer角色的配置文件中加入如下配置：

```cfg
peerType=observer
```

并在所有server的配置文件中，配置成observer模式的server的那行配置追加:observer，例如：

```cfg
server.3=192.168.60.130:2289:3389:observer
```











