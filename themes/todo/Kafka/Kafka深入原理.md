---
title: Kafka深入原理
date: 2020-01-31 09:29:18
categories:
	- Kafka
tags: 
	- Kafka
	- 深入原理
toc: true
---



### 1、主题

#### 1.1、操作主题相关的脚本（kafka-topics.sh）

主题表示一个消息的类别，每个消息都要归类到一个主题中去。还有分区，副本等概念。详细的解释请看《Kafka深入原理》。

##### 1.1.1、创建主题

在 kafka 根目录下执行如下命令：

```shell
bin/kafka-topics.sh --create --zookeeper 192.168.0.10:2181/kafka --topic first --partitions 3 --replication-factor 3
```

* **--create：**指定这是创建主题命令。
* **--zookeeper：**指定Zookeeper集群地址，最后还带上 /kafka 是因为在上面配置Kafka服务器的 zookeeper.connect 参数时，指定了根目录，所以这里也要带上。
* **--topic：**指定要创建的主题名称。
* **--partitions：**指定主题的分区数。
* **--replication-factor：**指定主题的副本数。



##### 1.1.2、查看主题

在 kafka 根目录下执行如下命令（查看主题中的分区数，副本数，分区Leader所在brokerID，副本列表，ISR列表）：

```shell
bin/kafka-topics.sh --describe --zookeeper 192.168.0.10:2181/kafka --topic first
```

* **--describe：**指定这是查看主题命令。
* **--zookeeper：**指定Zookeeper集群地址。
* **--topic：**指定要查看的主题名称。



##### 1.1.3、修改主题

在 kafka 根目录下执行如下命令：

```shell
bin/kafka-topics.sh --alter --zookeeper 192.168.0.10:2181/kafka --topic first --partitions 4
```

* **--alter：**指定这是修改主题命令。
* **--zookeeper：**指定Zookeeper集群地址。
* **--topic：**指定要修改的主题名称。
* **--partitions：**指定要修改的分区数。分区数只能增加，不能减少。



##### 1.1.4、查看主题列表

在 kafka 根目录下执行如下命令：

```shell
bin/kafka-topics.sh --list --zookeeper 192.168.0.10:2181/kafka
```

* **--list：**指定这是查看主题列表命令。
* **--zookeeper：**指定Zookeeper集群地址。



##### 1.1.5、删除主题

在 kafka 根目录下执行如下命令：

```shell
bin/kafka-topics.sh --delete --zookeeper 192.168.0.10:2181/kafka --topic first
```

* **--delete：**指定这是删除主题命令。
* **--zookeeper：**指定Zookeeper集群地址。
* **--topic：**指定要删除的主题名称。



#### 1.2、操作主题的Java客户端





### 2、分区管理

#### 2.1、分区副本分配



#### 2.2、优先副本的选举



#### 2.3、分区重分配



#### 2.4、复制限流



#### 2.5、修改副本因子



### 3、日志存储

#### 3.1、文件目录结构



#### 3.2、日志结构



#### 3.3、日志索引



#### 3.4、日志清理

##### 3.4.1、日志删除



##### 3.4.2、日志压缩



#### 3.5、日志存储

##### 3.5.1、顺序存储



##### 3.5.2、页缓存



##### 3.5.3、零拷贝



