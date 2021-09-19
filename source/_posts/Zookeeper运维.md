---
title: Zookeeper运维
date: 2020-04-28 15:03:04
categories:
	- Zookeeper
tags: 
	- Zookeeper
	- 运维
toc: true
typora-root-url: Zookeeper运维
---



> 本篇文章摘自《从Paxos到Zookeeper分布式一致性原理与实践》

### 1、四字命令

Zookeeper 中有很多4个英文字母长度的运维命令，简称为“四字命令”。

#### 1.1、使用方式

四字命令的使用方式非常简单，通常有两种方式。介绍如下：

* Telnet 方式

  实例:

  ```shell
  #连接
  telnet <ip> <port>
  
  #输入命令
  <命令>
  ```

  

* nc 方式

  ```shell
  echo <命令> | nc <ip> <port>
  ```

  如果没有安装，请先进行安装

  ```shell
  #root用户安装
  #下载安装包
  wget http://vault.centos.org/6.6/os/x86_64/Packages/nc-1.84-22.el6.x86_64.rpm
  #rpm安装
  rpm -iUv nc-1.84-22.el6.x86_64.rpm
  ```

  

#### 1.2、命令介绍

##### 1.2.1、conf

conf 命令用于输出 Zookeeper 服务器运行时使用的基本配置信息。

```shell
echo conf | nc localhost 2181
```

| 属性              | 说明                                                         |
| ----------------- | ------------------------------------------------------------ |
| clientPort        | 对外暴漏的客户端连接端口号。                                 |
| dataDir           | 数据快照文件目录，默认情况下100000次事务操作生成一次快照。   |
| dataLogDir        | 事务日志文件目录，生产环境中放在独立的磁盘上。               |
| tickTime          | 服务器之间或客户端与服务器之间维持心跳的时间间隔（以毫秒为单位）。 |
| maxClientCnxns    | 最大连接数。                                                 |
| minSessionTimeout | 最小session超时 minSessionTimeout=tickTime*2                 |
| maxSessionTimeout | 最大session超时 maxSessionTimeout=tickTime*20                |
| serverId          | 服务器编号。                                                 |
| initLimit         | 集群中的follower服务器(F)与leader服务器(L)之间初始连接时能容忍的最多心跳数。 |
| syncLimit         | 集群中的follower服务器(F)与leader服务器(L)之间 请求和应答之间能容忍的最多心跳数。 |
| electionAlg       | 0:基于UDP的LeaderElection 1:基于UDP的FastLeaderElection 2:基于UDP和认证的FastLeaderElection 3:基于TCP的FastLeaderElection 在3.4.10版本中，默认值为3另外三种算法已经被弃用，并且有计划在之后的版本中将它们彻底删除而不再支持。 |
| electionPort      | 选举端口。                                                   |
| quorumPort        | 数据通信端口。                                               |
| peerType          | 是否为观察者 1为观察者。                                     |



##### 1.2.2、cons

cons命令用于输出当前这台服务器上所有客户端连接的详细信息。

```shell
echo cons | nc localhost 2181
```

| 属性     | 说明                                                 |
| -------- | ---------------------------------------------------- |
| ip       | ip地址                                               |
| port     | 端口号                                               |
| queued   | 等待被处理的请求数，请求缓存在队列中                 |
| received | 收到的包数                                           |
| sent     | 发送的包数                                           |
| sid      | 会话id                                               |
| lop      | 最后的操作 GETD-读取数据 DELE-删除数据 CREA-创建数据 |
| est      | 连接时间戳                                           |
| to       | 超时时间                                             |
| lcxid    | 当前会话的操作id                                     |
| lzxid    | 最大事务id                                           |
| lresp    | 最后响应时间戳                                       |
| llat     | 最后/最新 延时                                       |
| minlat   | 最小延时                                             |
| maxlat   | 最大延时                                             |
| avglat   | 平均延时                                             |



##### 1.2.3、crst

crst 命令是一个功能性命令，用于重置所有的客户端连接统计信息。

```shell
echo crst | nc localhost 2181
```



##### 1.2.4、dump

dump 命令用于输出当前集群的所有会话信息，包括这些会话的会话ID，以及每个会话创建的临时节点等信息。

```shell
echo dump | nc localhost 2181
```

| 属性       | 说明                                                   |
| ---------- | ------------------------------------------------------ |
| session id | znode path（1对多，处于队列中排队的session和临时节点） |



##### 1.2.5、envi

envi 命令用于输出 Zookeeper 所在服务器运行时的环境信息。

```shell
echo envi | nc localhost 2181
```

| 属性              | 说明                                        |
| ----------------- | ------------------------------------------- |
| zookeeper.version | 版本                                        |
| host.name         | host信息                                    |
| java.version      | java版本                                    |
| java.vendor       | 供应商                                      |
| java.home         | 运行环境所在目录                            |
| java.class.path   | classpath                                   |
| java.library.path | 第三方库指定非java类包的位置（如：dll，so） |
| java.io.tmpdir    | 默认的临时文件路径                          |
| java.compiler     | JIT 编译器的名称                            |
| os.name           | Linux                                       |
| os.arch           | amd64                                       |
| os.version        | 3.10.0-514.el7.x86_64                       |
| user.name         | zookeeper                                   |
| user.home         | 用户名目录                                  |
| user.dir          | 服务所在目录                                |



##### 1.2.6、ruok

ruok 命令用于输出当前 Zookeeper 服务器是否正在运行。

```shell
echo ruok | nc localhost 2181
```

如果当前 Zookeeper 服务器正在运行，那么返回 “imok” ，否则没有任何响应输出。



##### 1.2.7、stat

stat 命令用于获取 Zookeeper 服务器的运行时状态信息。

```shell
echo stat | nc localhost 2181
```

| 属性                | 说明           |
| ------------------- | -------------- |
| Zookeeper version   | 版本           |
| Clients             | 客户端连接列表 |
| Latency min/avg/max | 延时           |
| Received            | 收包           |
| Sent                | 发包           |
| Connections         | 连接数         |
| Outstanding         | 堆积数         |
| Zxid                | 最大事物id     |
| Mode                | 服务器角色     |
| Node                | 节点数         |



##### 1.2.8、srvr

srvr 命令和 stat 命令的功能一致，唯一的区别是 srvr 不会将客户端的连接情况输出，仅仅输出服务器的自身信息。



##### 1.2.9、srst

srst 命令是一个功能行命令，用于重置所有服务器的统计信息。

```shell
echo srst | nc localhost 2181
```



##### 1.2.10、wchs

wchs 命令用于输出当前服务器上管理的 Watcher 的概要信息。

```shell
echo wchs | nc localhost 2181
```

| 属性         | 说明        |
| ------------ | ----------- |
| connectsions | 连接数      |
| watch-paths  | watch节点数 |
| watchers     | watcher数量 |



##### 1.2.11、wchc

wchc 命令用于输出当前服务器上管理的 Watcher 的详细信息，以会话为单位进行归组，同时列出被该会话注册了 Watcher 的节点路径。

```shell
echo wchc | nc localhost 2181
```

问题:

```shell
wchc is not executed because it is not in the whitelist.
```

解决方法：

```sh
# 修改启动指令 zkServer.sh 
​
# 注意找到这个信息
else
    echo "JMX disabled by user request" >&2
    ZOOMAIN="org.apache.zookeeper.server.quorum.QuorumPeerMain" 
fi
​
# 下面添加如下信息
ZOOMAIN="-Dzookeeper.4lw.commands.whitelist=* ${ZOOMAIN}"
```



##### 1.2.12、wchp

wchp 命令和 wchc 命令非常类似，也是用于输出当前服务器上管理的 Watcher 的详细信息，不同点在于 wchp 命令的输出信息以节点路径为单位进行归组。

```shell
echo wchp | nc localhost 2181
```



##### 1.2.13、mntr

mntr 命令用于输出比 stat 命令更为详尽的服务器统计信息。

| 属性                            | 说明                 |
| ------------------------------- | -------------------- |
| zk_version                      | 版本                 |
| zk_avg_latency                  | 平均延时             |
| zk_max_latency                  | 最大延时             |
| zk_min_latency                  | 最小延时             |
| zk_packets_received             | 收包数               |
| zk_packets_sent                 | 发包数               |
| zk_num_alive_connections        | 连接数               |
| zk_outstanding_requests         | 堆积请求数           |
| zk_server_state                 | leader/follower 状态 |
| zk_znode_count                  | znode数量            |
| zk_watch_count                  | watch数量            |
| zk_ephemerals_count             | 临时节点（znode）    |
| zk_approximate_data_size        | 数据大小             |
| zk_open_file_descriptor_count   | 打开的文件描述符数量 |
| zk_max_file_descriptor_count    | 最大文件描述符数量   |
| zk_fsync_threshold_exceed_count | 0                    |



### 2、Zookeeper 图形化的客户端工具（ZooInspector）

ZooInspector下载地址：

```
https://issues.apache.org/jira/secure/attachment/12436620/ZooInspector.zip
```

解压后进入目录ZooInspector\build，运行zookeeper-dev-ZooInspector.jar

```
#执行命令如下
java -jar zookeeper-dev-ZooInspector.jar
```

![](运行.JPG)

点击左上角连接按钮，输入zk服务地址：ip或者主机名:2181

![](连接.JPG)

点击OK，即可查看ZK节点信息

![](连接成功.JPG)



### 3、taoKeeper 监控工具

基于zookeeper的监控管理工具taokeeper，由淘宝团队开源的zk管理中间件，安装前要求服务前先配置nc 和 sshd

1.下载数据库脚本

```
wget https://github.com/downloads/alibaba/taokeeper/taokeeper.sql
```

2.下载主程序

```
wget https://github.com/downloads/alibaba/taokeeper/taokeeper-monitor.tar.gz
```

3.下载配置文件

```
wget https://github.com/downloads/alibaba/taokeeper/taokeeper-monitor-config.properties
```

4.配置 taokeeper-monitor-config.properties

```
#Daily
systemInfo.envName=DAILY
#DBCP
dbcp.driverClassName=com.mysql.jdbc.Driver
#mysql连接的ip地址端口号
dbcp.dbJDBCUrl=jdbc:mysql://localhost:3306/taokeeper
dbcp.characterEncoding=GBK
#用户名
dbcp.username=root
#密码
dbcp.password=root
dbcp.maxActive=30
dbcp.maxIdle=10
dbcp.maxWait=10000
#SystemConstant
#用户存储内部数据的文件夹
#创建/home/zookeeper/taokeeperdata/ZooKeeperClientThroughputStat
SystemConstent.dataStoreBasePath=/home/zookeeper/taokeeperdata
#ssh用户
SystemConstant.userNameOfSSH=zookeeper
#ssh密码
SystemConstant.passwordOfSSH=zookeeper
#Optional
SystemConstant.portOfSSH=22
```

5.安装配置 tomcat，修改catalina.sh

```
#指向配置文件所在的位置
JAVA_OPTS=-DconfigFilePath="/home/zookeeper/taokeeper-monitor-tomcat/webapps/ROOT/conf/taokeeper-monitor-config.properties"
```

6.部署工程启动