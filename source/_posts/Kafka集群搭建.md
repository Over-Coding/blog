---
title: Kafka集群搭建
date: 2020-05-09 09:10:44
categories:
	- Kafka
tags: 
	- Kafka
	- 集群搭建
toc: true
---



### 1、集群规划

| IP地址          | 系统    | 环境                   |
| --------------- | ------- | ---------------------- |
| 192.168.150.101 | CentOS7 | jdk8，Zookeeper 3.4.14 |
| 192.168.150.102 | CentOS7 | jdk8，Zookeeper 3.4.14 |
| 192.168.150.103 | CentOS7 | jdk8，Zookeeper 3.4.14 |



### 2、下载安装

#### 2.1、下载

下载地址：http://kafka.apache.org/downloads

根据自己的需求，选择相对应的版本，我这是用的是kafka_2.11-1.1.1。

**解释：kafka_2.11-1.1.1中有两个版本号，2.11表示Scala的版本号，1.1.1表示Kafka的版本号。**



#### 2.2、安装

将下载好的压缩包，解压到自己指定的路径下，例如：

```shell
tar -zxvf kafka_2.11-1.1.1.tgz -C /opt/module/
```



### 3、配置文件参数说明

进入到 config/ 目录下，里面有一个 server.properties 文件，这个就是 kafka 的配置文件。这里只介绍部分参数，其他参数请看 http://kafka.apache.org/documentation/#brokerconfigs

| 参数名                          | 默认值          | 描述                                                         |
| :------------------------------ | :-------------- | :----------------------------------------------------------- |
| broker.id                       | -1              | 表示该节点在Kafka集群中broker的唯一标识。                    |
| listeners                       | 无              | 表示broker 监听客户端连接的地址列表。配置格式为：protocol://hostname:port。其中protocol 表示协议类型，hostname表示主机名，port表示端口号。 |
| log.dirs                        | /tmp/kafka-logs | 表示用来配置Kafka日志文件存放的根目录。并且可以配置多个根目录（以逗号分隔）。 |
| num.partitions                  | 1               | 表示主题的分区数。                                           |
| log.retention.hours             | 168             | 表示日志文件的留存时间，单位为小时。                         |
| log.segment.bytes               | 1073741824      | 表示日志分段文件的最大值，超过这个值会强制创建一个新的日志分段。 |
| zookeeper.connect               | 无              | 表示连接的Zookeeper集群地址，表示多个地址时用逗号分隔。并且可以配置chroot路径，即指定节点为根路径。例如：192.168.150.101:2181,192.168.150.102:2181, 192.168.150.103:2181/kafka |
| zookeeper.connection.timeout.ms | 6000            | 表示连接 Zookeeper 集群的超时时间。                          |
| delete.topic.enable             | true            | 表示是否开启删除主题。                                       |



### 4、修改配置

#### 4.1、修改配置文件

修改以下参数：

```cfg
broker.id=0
listeners=PLAINTEXT://192.168.0.10:9092
log.dirs=/data/kafka/data
zookeeper.connect=192.168.150.101:2181,192.168.150.102:2181,192.168.150.103:2181/kafka
```

**注意：这里只进行了简单的配置，请根据自己的业务需求进行修改。**



#### 4.2、创建数据目录

根据配置文件中的 log.dirs 参数创建对应的目录。



### 5、配置其他机器

根据上面的步骤在其他两台机器上进行配置。

**注意：一定要根据不同服务器修改 broker.id 和 listeners 参数。**



### 6、启动与关闭服务

在根目录中有一个 /bin 文件夹，Kafka中所有的脚本工具都在这个目录下。

#### 6.1、启动服务

在 kafka 根目录下执行如下命令：

```shell
bin/kafka-server-start.sh -daemon config/server.properties
```

* -daemon： 表示指定服务后台运行。

**注意：在启动 kafka 集群之前，要先启动 Zookeeper 集群。**



#### 6.2、关闭服务

在 kafka 根目录下执行如下命令：

```shell
bin/kafka-server-stop.sh
```











