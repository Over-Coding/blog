---
title: Zookeeper客户端使用
date: 2020-04-12 09:15:08
categories:
	- Zookeeper
tags: 
	- Zookeeper
	- 客户端
toc: true
---



> 本篇文章借鉴了《从Paxos到Zookeeper分布式一致性原理与实践》

本篇将会介绍Zookeeper自带的客户端脚本，Java客户端API和开源客户端。



### 1、客户端脚本

在Zookeeper的安装目录下的 **/bin** 文件夹中有一个 **zkCli.sh** 脚本，这个是官方提供的客户端脚本。

#### 1.1、连接

可以直接运行zkCli.sh脚本，默认就是连接本地的Zookeeper服务器。

```shell
sh zkCli.sh
```

如果想连接指定的Zookeeper服务器，需要在后面添加一些参数。

```shell
sh zkCli.sh -server ip:port
```

* -server：表示指定Zookeeper服务器，后面跟服务器的IP地址和端口号。



**例子：**

指定连接 192.168.0.10 上的 Zookeeper 服务器。

```shell
sh zkCli.sh -server 192.168.0.10:2181
```



#### 1.2、创建

使用 create 命令可以创建一个 ZNode 节点。默认创建永久节点。用法如下：

```shell
create [-s] [-e] path data acl
```

* -s：表示创建一个顺序节点。
* -e：表示创建一个临时节点。
* path：表示要创建的节点路径名。
* data：表示要创建的节点数据。
* acl：表示要创建的节点的访问控制列表。可以不写，默认为 world:anyone:cdrwa。



**注意：**

* 不可以递归创建。例如：创建 /create/node 节点时，其父节点 /create 必须已经存在。
* 临时节点下不可以创建子节点。



**例子：**

创建一个永久顺序节点，节点路径为 **/node**，节点数据为 **node**，访问控制列表为 ip:192.168.0.10:cdrwa**。

```shell
create -s /node "node" ip:192.168.0.10:cdrwa
```



#### 1.3、读取

##### 1.3.1、ls

使用 ls 命令可以列出 Zookeeper 指定节点下的所有下级节点。用法如下：

```shell
ls path [watch]
```

* path：表示要查询的节点路径。
* watch：表示是否开启监控，监控 path 下的子节点变化（NodeChildrenChanged，NodeDeleted）。



**例子：**

列出根节点下的所有子节点。**/** 表示根节点。

```shell
ls /
```



##### 1.3.3、ls2

使用 ls2 命令可以列出 Zookeeper 指定节点下的所有下级节点，并且显示当前节点的属性信息。用法如下：

```shell
ls2 path [watch]
```

* path：表示要查询的节点路径。
* watch：表示是否开启监控，监控 path 下的子节点变化。



**例子：**

列出根节点下的所有子节点，并且查看根节点的属性信息。**/** 表示根节点。

```shell
ls2 /
```



##### 1.3.2、get

使用 get 命令可以获取 Zookeeper 指定节点的数据内容和属性信息。用法如下：

```shell
get path [watch]
```

* path：表示要查询的节点路径。
* watch：表示是否开启监控，监控节点变化（NodeDataChanged，NodeDeleted）。



**例子：**

获取 /node 节点的数据和属性信息。

```shell
get /node
```



#### 1.4、更新

使用 set 命令可以更新指定节点的数据内容。用法如下：

```shell
set path data [version]
```

* path：表示要更新的节点路径名。
* data：表示要更新的节点数据。
* version：表示指定基于哪个节点数据版本进行更新。



**例子：**

更新 /node 节点数据为 test。

```shell
set /node "test"
```



#### 1.5、删除

##### 1.5.1、delete

使用 delete 命令可以删除 Zookeeper 上的指定节点。用法如下：

```shell
delete path [version]
```

* path：表示要删除的节点路径名。
* version：表示指定基于哪个节点数据版本进行删除。



**注意：**不支持递归删除。例如：删除 /create 节点时，其节点下不能存在子节点。



**例子：**

删除 /node 节点。

```shell
delete /node
```



##### 1.5.2、rmr

使用 rmr 命令可以递归删除 Zookeeper 上的指定节点和其所有的子节点。用法如下：

```shell
rmr path
```

* path：表示要删除的节点路径名。



**例子：**

删除 /node 节点和其下的所有子节点。

```shell
rmr /node
```



#### 1.6、ACL

##### 1.6.1、setAcl

使用 setAcl 命令可以设置 Zookeeper 上的指定节点的访问控制列表，即访问权限。具体可以设置哪些访问权限，请阅读 Zookeeper深入原理。用法如下：

```shell
setAcl path acl
```

* path：表示要设置权限的节点路径名。
* acl：表示要设置的权限列表，可以设置多种权限。



**例子：**

为 /node 节点设置权限 **ip:192.168.0.10:cdrwa** 。

```shell
setAcl /node ip:192.168.0.10:cdrwa
```



##### 1.6.2、getAcl

使用 getAcl 命令可以获取 Zookeeper 上的指定节点的访问控制列表。用法如下：

```shell
getAcl path
```

* path：表示要获取权限的节点路径名。



**例子：**

获取 /node 节点的访问控制列表。

```shell
getAcl /node
```



#### 1.7、其他

##### 1.7.1、stat

使用 stat 命令可以获取 Zookeeper 上的指定节点的属性信息。用法如下：

```shell
stat path [watch]
```

* path：表示要获取属性信息的节点路径名。
* watch：表示是否开启监控，监控节点变化（NodeDataChanged，NodeDeleted）。



**例子：**

查看 /node 节点属性信息。

```shell
stat /node
```



##### 1.7.2、connect

使用 connect 命令可以连接 Zookeeper 上的指定服务器。用法如下：

```shell
connect host:port
```

* host：表示要连接的 Zookeeper 服务器IP。
* port：表示要连接的 Zookeeper 服务器端口号。



**例子：**

连接 192.168.0.12:2181 Zookeeper 服务器。

```shell
connect 192.168.0.12:2181
```



##### 1.7.3、close

使用 close 命令可以关闭当前会话。用法如下：

```shell
close
```



##### 1.7.4、quit

使用 quit 命令可以退出当前客户端。用法如下：

```shell
quit
```



### 2、Java客户端API

Zookeeper 官方提供了很多编程语言的客户端API，这里我只介绍Java客户端API。

API地址：https://zookeeper.apache.org/doc/r3.4.14/api/index.html

#### 2.1、创建会话

客户端可以通过创建一个 Zookeeper（org.apache.zookeeper.Zookeeper） 实例来连接 Zookeeper服务器。

**API列表：**

* **Zookeeper**(String connectString, int sessionTimeout, Watcher watcher)
* **Zookeeper**(String connectString, int sessionTimeout, Watcher watcher, boolean canBeReadOnly)
* **Zookeeper**(String connectString, int sessionTimeout, Watcher watcher, long sessionId, byte[] sessionPasswd)
* **Zookeeper**(String connectString, int sessionTimeout, Watcher watcher, long sessionId, byte[] sessionPasswd, boolean canBeReadOnly)



**参数介绍：**

| 参数名                     | 说明                                                         |
| -------------------------- | ------------------------------------------------------------ |
| connectString              | Zookeeper服务器连接地址，多个服务器可以使用 **,** 连接。例如：192.168.0.10:2181,192.168.0.11:2181,192.168.0.12:2181 |
| sessionTimeout             | 指会话的超时时间，单位是毫秒。                               |
| watcher                    | Zookeeper允许客户端在构造方法中传入一个接口 Watcher 的实现类对象来作为默认的 Watcher 事件通知处理器。该参数也可以设置为 null，表明不设置默认的 Watcher 处理器。 |
| canBeReadOnly              | 这是一个 boolean 类型的参数，用于标识当前会话是否支持只读模式。 |
| sessionId 和 sessionPasswd | 分别代表会话ID 和 会话密钥。这两个参数能够唯一确定一个会话，同时客户端可以使用这两个参数实现客户端会话复用，从而达到恢复会话的效果。 |



**例子：**

注意：Zookeeper客户端和服务器会话的建立是一个异步过程。

```java
public class ZookeeperConnect {
    //阻塞主线程，等待会话创建成功
	private CountDownLatch countDownLatch = new CountDownLatch(1);
    
    public static void main(String[] args) throws Exception{
        //创建Zookeeper对象，连接服务器
		ZooKeeper zooKeeper = new ZooKeeper("192.168.0.10:2181", 5000, new Watcher() {
            @Override
            public void process(WatchedEvent watchedEvent) {
                //连接成功
                if(Event.KeeperState.SyncConnected.equals(watchedEvent.getState())){
                    countDownLatch.countDown();
                }else if(Event.KeeperState.Disconnected.equals(watchedEvent.getState())){
                    System.out.println("连接失败......");
                }else if(Event.KeeperState.AuthFailed.equals(watchedEvent.getState())){
                    System.out.println("用户认证失败......");
                }
            }
        });
        //等待连接成功
        countDownLatch.await();
        System.out.println("连接成功......");
    }
}
```



#### 2.2、创建节点

客户端可以通过Zookeeper的API来创建一个数据节点。

**API列表：**

* String  **create**(String path, byte[] data, List<ACL> acl, CreateMode createMode)
* void **create**(String path, byte[] data, List<ACL> acl, CreateMode createMode, AsyncCallback.StringCallback cb, Object ctx)



**参数介绍：**

| 参数名     | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| path       | 要创建的节点路径。                                           |
| data       | 要创建的节点的数据内容。                                     |
| acl        | 要创建的节点访问控制列表。                                   |
| createMode | 要创建的节点的节点类型。节点类型：持久，持久顺序，临时，临时顺序。 |
| cb         | 回调函数，用于异步创建时使用。                               |
| ctx        | 上下文信息，用于异步创建时传递数据使用。                     |



#### 2.3、读取数据

读取数据包括节点数据的获取和子节点列表的获取。

##### 2.3.1、getData

客户端可以通过 Zookeeper 的 API 来获取一个节点的数据内容。

**API列表：**

* byte[] **getData**(String path, boolean watch, Stat stat)
* byte[] **getData**(String path, Watcher watcher, Stat stat)
* void **getData**(String path, boolean watch, AsyncCallback.DataCallback cb, Object ctx)
* void **getData**(java.lang.String path, Watcher watcher, AsyncCallback.DataCallback cb, Object ctx)



**参数介绍：**

| 参数名  | 说明                                                       |
| ------- | ---------------------------------------------------------- |
| path    | 指定数据节点的节点路径                                     |
| watch   | 表明是否需要注册一个 Watcher。如果是就使用默认的 Watcher。 |
| stat    | 指定数据节点的状态信息。                                   |
| watcher | 注册Watcher。                                              |
| cb      | 回调函数，用于异步获取数据时使用。                         |
| ctx     | 上下文信息，用于异步获取数据时传递数据使用。               |



##### 2.3.2、getChildren

客户端可以通过 Zookeeper 的 API 来获取一个节点的所有子节点。

**API列表：**

* List<String> **getChildren**(String path, boolean watch)
* void **getChildren**(String path, boolean watch, AsyncCallback.Children2Callback cb, Object ctx)
* void **getChildren**(String path, boolean watch, AsyncCallback.ChildrenCallback cb, Object ctx)
* List<String> **getChildren**(String path, boolean watch, Stat stat)
* List<String> **getChildren**(String path, Watcher watcher)
* void **getChildren**(String path, Watcher watcher, AsyncCallback.Children2Callback cb, Object ctx)
* void **getChildren**(String path, Watcher watcher, AsyncCallback.ChildrenCallback cb, Object ctx)
* List<String> **getChildren**(String path, Watcher watcher, Stat stat)



**参数介绍：**

| 参数名  | 说明                                                       |
| ------- | ---------------------------------------------------------- |
| path    | 指定数据节点的节点路径。                                   |
| watch   | 表明是否需要注册一个 Watcher。如果是就使用默认的 Watcher。 |
| watcher | 注册Watcher。                                              |
| cb      | 回调函数，用于异步获取数据时使用。                         |
| ctx     | 上下文信息，用于异步获取数据时传递数据使用。               |
| stat    | 指定数据节点的节点状态信息。                               |



#### 2.4、更新数据

客户端可以通过 Zookeeper 的 API 来更新一个节点的数据内容。

**API列表：**

* Stat **setData**(String path, byte[] data, int version)
* void **setData**(String path, byte[] data, int version, AsyncCallback.StatCallback cb, Object ctx)



**参数介绍：**

| 参数名  | 说明                                                         |
| ------- | ------------------------------------------------------------ |
| path    | 指定数据节点的节点路径。                                     |
| data[]  | 一个字节数组，即需要使用该数据内容来覆盖节点现在的数据内容。 |
| version | 指定节点的数据版本。                                         |
| cb      | 回调函数，用于异步更新数据时使用。                           |
| ctx     | 上下文信息，用于异步更新数据时传递数据使用。                 |



#### 2.5、删除节点

客户端可以通过Zookeeper的API来删除一个数据节点。

**API列表：**

* void **delete**(String path, int version)
* void **delete**(String path, int version, AsyncCallback.VoidCallback cb, Object ctx)



**参数介绍：**

| 参数名  | 说明                                     |
| ------- | ---------------------------------------- |
| path    | 要删除的节点路径                         |
| version | 指定节点的数据版本                       |
| cb      | 回调函数，用于异步删除时使用。           |
| ctx     | 上下文信息，用于异步删除时传递数据使用。 |



#### 2.6、检测节点是否存在

客户端可以通过 Zookeeper 的 API 来检测节点是否存在。

**API列表：**

* State **exists**(String path, boolean watch)
* State **exists**(String path, Watcher watcher)
* void **exists**(String path, boolean watch, AsyncCallback.StatCallback cb, Object ctx)
* void **exists**(String path, Watcher watcher, AsyncCallback.StatCallback cb, Object ctx)



**参数介绍：**

| 参数名  | 说明                                                         |
| ------- | ------------------------------------------------------------ |
| path    | 指定数据节点的节点路径。                                     |
| watch   | 指定是否复用 Zookeeper 中默认的 Watcher。                    |
| watcher | 注册的Watcher，用于监听以下三类事件：节点被创建、节点被删除、节点被更新 |
| cb      | 回调函数，用于异步检测节点是否存在时使用。                   |
| ctx     | 上下文信息，用于异步检测节点是否存在时传递数据使用。         |



### 3、开源客户端

#### 3.1、ZkClient

ZkClient 是 Github 上一个开源的 Zookeeper 客户端，是由 Datameer 的工程师 Stefan Groschupf 和 Peter Voss 一起开发的。ZkClient 在 Zookeeper 原生 API 接口之上进行了包装，是一个更易用的Zookeeper 客户端。同时，ZkClient 在内部实现了诸如 Session 超时重连、Watcher 反复注册等功能，使得 Zookeeper 客户端的这些繁琐的细节工作对开发人员透明。

这里我就不演示使用方法了，自己去探索研究吧。



#### 3.2、Curator

Curator 是 Netflix 公司开源的一套 Zookeeper 客户端框架，其作者是 Jordan Zimmerman。和 ZkClient 一样，Curator 解决了很多 Zookeeper 客户端非常底层的细节开发工作，包括连接重连、反复注册 Watcher 和 NodeExistsException 异常等，目前已经成为了 Apache 的顶级项目，是全世界范围内使用最广泛的 Zookeeper 客户端之一。并且，Curator 在 Zookeeper 原生API的基础上进行了包装，提供了一套易用性和可读性更强的 Fluent 风格的客户端 API 框架。

除此之外，Curator 中还提供了 Zookeeper 各种应用场景（Recipe，如共享锁服务、Master选举机制和分布式计数器等）的抽象封装。

官网地址：http://curator.apache.org/

同样，这里我就不演示使用方法了，自己去探索研究吧。



源码地址：https://gitee.com/lingfeng1024/stu-zookeeper
