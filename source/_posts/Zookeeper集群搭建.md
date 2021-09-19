---
title: Zookeeper集群搭建
date: 2020-04-11 01:12:44
categories:
	- Zookeeper
tags: 
	- Zookeeper
	- 集群搭建
toc: true
---



> 本篇文章借鉴了《从Paxos到Zookeeper分布式一致性原理与实践》

Zookeeper有两种运行模式：单机模式和集群模式。因为单机模式只是在开发测试时使用，所以这里就不介绍单机模式的搭建。



### 1、集群规划

注意：因为Zookeeper遵循半数原则，所以集群节点个数最好是奇数。

| IP地址       | 系统    | 环境 |
| ------------ | ------- | ---- |
| 192.168.0.10 | CentOS7 | jdk8 |
| 192.168.0.11 | CentOS7 | jdk8 |
| 192.168.0.12 | CentOS7 | jdk8 |



### 2、下载安装

#### 2.1、下载

下载地址：http://archive.apache.org/dist/zookeeper/

根据自己的需求，选择相对应的版本，我这是用的是zookeeper-3.4.14。



#### 2.2、安装

将下载好的压缩包，解压到自己指定的路径下，例如：

```shell
tar -zxvf zookeeper-3.4.14.tar.gz -C /opt/module/
```



### 3、配置文件参数说明

进入到 conf/ 目录下，里面有一个 zoo_sample.cfg 文件，这个就是zookeeper的配置文件。我们先对里面的参数介绍下：

| 参数名                   | 默认值  | 描述                                                         |
| :----------------------- | :------ | :----------------------------------------------------------- |
| tickTime                 | 2000    | 单位是毫秒（ms）。表示zookeeper中的最小时间单元的长度，其他参数都是以tickTime为单位来配置。 |
| initLimit                | 10      | 单位是tickTime。表示允许Follower初始化连接到Leader并完成数据同步的超时时间，如果ZooKeeper管理的数据量很大，请根据需要增加此值。 |
| syncLimit                | 5       | 单位是tickTime。表示Leader与Follower之间进行心跳检测的最大延时时间。如果超过时间，那么Leader认为该Follower已经脱离了集群。 |
| dataDir                  | 无      | **必须配置。**用于配置Zookeeper服务器存储快照文件的目录。    |
| dataLogDir               | dataDir | 用于配置Zookeeper服务器存储事务日志文件的目录。推荐配置为单独的一个磁盘上，因为事务日志写入的性能直接决定了Zookeeper在处理事务请求时的吞吐。 |
| clientPort               | 2181    | **必须配置。**用于配置当前服务器对外的服务端口，客户端会通过该端口和Zookeeper服务器创建连接。 |
| snapCount                | 100000  | 用于配置相邻两次数据快照之间的事务操作次数，即Zookeeper会在snapCount次事务操作之后进行一次数据快照。 |
| preAllocSize             | 65536   | 单位是KB。用于配置Zookeeper事务日志文件预分配的磁盘空间大小。 |
| minSessionTimeout        | 2       | 单位是tickTime。用于配置服务端和客户端会话超时时间的最小值。 |
| maxSessionTimeout        | 20      | 单位是tickTime。用于配置服务端和客户端会话超时时间的最大值。 |
| maxClientCnxns           | 60      | 用于从Socket层面限制单个客户端与单台服务器之间的并发连接数，即以IP地址粒度来进行连接数的限制。如果将该参数设置为0，则表示对连接数不作任何限制。 |
| server.id=host:port:port | 无      | 用于配置组成Zookeeper集群的机器列表,其中id即为ServerID，与每台服务器myid文件中的数字相对应。同时,在该参数中，会配置两个端口：第一个端口用于指定Follower服务器与Leader进行运行时通信和数据同步时所使用的端口，第二个端口则专门用于进行Leader选举过程中的投票通信。 |
| autopurge.snapRetainCont | 3       | 用于配置Zookeeper在自动清理的时候需要保留的快照数据文件数量和对应的事务日志文件。需要注意的是，并不是磁盘上的所有事务日志和快照数据文件都可以被清理掉——那样的话将无法恢复数据。因此 autopurge.snapRetainCont 的最小值是3，如果配置的 autopurge.snapRetainCont 值比3小的话，那么会被自动调整到3，即至少需要保留3个快照数据文件和对应的事务日志文件。 |
| autopurge.purgeInterval  | 0       | 单位是小时。用于配置Zookeeper进行历史文件自动清理的频率。如果配置该值为0或负数，那么就表明不需要开启定时清理功能。 |



### 4、修改配置

#### 4.1、修改配置文件

先将 zoo_sample.cfg 文件重命名为zoo.cfg，然后修改里面的参数，我修改的参数如下：

```cfg
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/data/zookeeper/data
clientPort=2181
server.1=192.168.0.10:2888:3888
server.2=192.168.0.11:2888:3888
server.3=192.168.0.12:2888:3888
```

**注意：这里只进行了简单的配置，请根据自己的业务需求进行修改。**

#### 4.2、创建数据目录

根据配置文件中的dataDir参数创建对应的目录，然后在此目录下创建名为myid的文件，添加配置的当前节点id。



### 5、配置其他机器

根据上面的步骤在其他两台机器上进行配置。

**注意：一定要修改myid文件中的数据为对应节点的id。**



### 6、启动与关闭服务

在根目录中有一个 /bin 文件夹，zookeeper中所有的命令都在这个目录下。

#### 6.1、启动服务

```shell
./zkServer.sh start
```



#### 6.2、查看服务状态

```shell
./zkServer.sh status
```



#### 6.2、关闭服务

```shell
./zkServer.sh stop
```
