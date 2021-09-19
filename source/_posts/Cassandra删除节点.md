---
title: Cassandra删除节点
date: 2020-02-20 01:12:44
categories:
	- Cassandra
tags: 
	- Cassandra
toc: true
---

### 1、删除在线节点

在要删除的节点服务器上，执行命令：

```shell
nodetool decommission
```
然后等待同步数据。



### 2、删除离线节点

#### 2.1、查看集群所有节点状态

在一个在线节点服务器上执行命令：

```shell
#查看集群所有节点
nodetool status
```


#### 2.2、删除节点

删除指定节点，执行命令：

```shell
nodetool removenode [ndoeId]
```
等待时间可能有点长，如果长时间没反应，就kill 掉上面的命名，然后执行：

```shell
nodetool removenode force
```


#### 2.3、重新查看节点状态

再查看节点状态，应该就删除成功了。

```shell
#查看集群所有节点
nodetool status
```