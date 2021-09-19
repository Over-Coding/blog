---
title: Linux下安装Elasticsearch
date: 2020-02-09 16:08:31
categories:
	- Elasticsearch
tags:
	- Elasticsearch
	- 集群搭建
toc: true
typora-root-url: Elasticsearch-install-on-Linux
---
### 1、搭建集群
#### 1.1、准备环境
* Linux服务器
* jdk8以上



#### 1.2、下载并安装

下载地址：https://www.elastic.co/cn/downloads/elasticsearch

执行解压安装命令

```shell
tar -zxvf elasticsearch-6.3.2.tar.gz -C /opt/module/
```



#### 1.3、配置

进入到 **/opt/module/elasticsearch-6.3.2/config** 目录下，修改配置文件**elasticsearch.yml**

```yml
#集群名称，一个集群的所有节点的集群名称必须相同
cluster.name: es-test
#节点名称，每个节点之间都不能相同
node.name: node-101
#数据存放路径
path.data: /data/elasticsearch/data
#日志存放路径
path.logs: /data/elasticsearch/logs
#当前节点ip地址
network.host: 192.168.150.101
#network.host: 0.0.0.0
#http访问端口
http.port: 9200
#集群访问端口
transport.tcp.port: 9300
#是否为主节点
node.master: true
#是否为数据节点
node.data: true
#集群其他节点地址
discovery.zen.ping.unicast.hosts: ["192.168.150.102","192.168.150.103"]
#最少主节点数
discovery.zen.minimum_master_nodes: 2
```
其他节点也是同样配置，不过配置文件有少许改动，请按自己的集群进行配置。



#### 1.4、启动

进入到 **/opt/module/elasticsearch-6.3.2/bin** 执行如下命令：

```shell
./elasticsearch
```
后台启动：

```shell
./elasticsearch -d
```



#### 1.5、解决启动错误

![](1.png)
* [1]和[2]问题解决方案：

修改 **/etc/security/limits.conf** ,在文件结束之前添加如下配置：

```conf
#打开文件的最大数目
* hard nofile 655360
#进程的最大数目
* soft nofile 131072
#当前系统生效的设置值
* hard nproc 4096
#系统所能设定的最大值
* soft nproc 2048
```
* [3]解决方案：

修改 **/etc/sysctl.conf** ,在文件的最后添加如下配置：

```conf
#在缺省配置下，单个jvm能开启的最大线程数为其一半
vm.max_map_count=655360
#设置系统所有进程一共可以打开的文件数量
fs.file-max=655360
```
然后执行如下命令：

```shell
sysctl -p
```
**注意：如果启动es还是报错，请重启一下服务器。**



#### 1.6、测试

使用浏览器访问：

```url
http://192.168.150.101:9200/_cat/nodes/?v
```
出现如下页面，说明部署成功。

![](2.PNG)

### 2、可视化工具
#### 2.1、Kibana
##### 2.1.1、下载

地址：https://www.elastic.co/cn/downloads/past-releases#kibana

注意：要选与elasticsearch对应的版本



##### 2.1.2、解压安装

```shell
tar -zxvf kibana-6.3.2-linux-x86_64.tar.gz -C /opt/module/
```


##### 2.1.3、配置

进入到 **/opt/module/kibana-6.3.2-linux-x86_64/config** 目录下，修改配置文件 **kibana.yml** 

```yml
#端口号
server.port: 5601
#服务器ip地址
server.host: "192.168.150.101"
#连接的elasticsearch url地址
elasticsearch.url: "http://192.168.150.101:9200"
```


##### 2.1.4、启动

进入到 **/opt/module/kibana-6.3.2-linux-x86_64/bin** 目录下，执行如下命令：

```shell
./kibana
```
后台启动：

```shell
nohup ./kibana &
```


##### 2.1.5、连接

使用浏览器访问：

```url
http://192.168.150.101:5601
```



#### 2.2、cerebro

##### 2.2.1、下载

地址：https://github.com/lmenezes/cerebro/releases



##### 2.2.2、解压安装

```shell
tar -zxvf cerebro-0.8.5.tgz -C /opt/module/
```


##### 2.2.3、配置

进入到 **/opt/module/cerebro-0.8.5/conf** 目录下，修改配置文件 **application.conf** 

```yml
#服务运行的pid存放位置，如要避免产生pid可使用/dev/null
pidfile.path=/dev/null
#cerebro 存储数据的位置，默认为cerebro安装目录
data.path = "./cerebro.db"
#配置需要连接的es集群
hosts = [
  {
     # 集群连接地址
     host = "http://192.168.150.101:9200"
     #自定义集群名称
     name = "Weather cluster"
  }

  #{
  #  host = "http://localhost:9200"
  #  name = "Localhost cluster"
  #  headers-whitelist = [ "x-proxy-user", "x-proxy-roles", "X-Forwarded-For" ]
  #}
  # Example of host with authentication
  #{
  #  host = "http://some-authenticated-host:9200"
  #  name = "Secured Cluster"
  #  auth = {
  #    username = "username"
  #    password = "secret-password"
  #  }
  #}
]

```


##### 2.2.4、启动

进入到 **/opt/module/cerebro-0.8.5/bin** 目录下，执行如下命令：

```shell
./cerebro
```
后台启动：

```shell
nohup ./cerebro &
```
后台启动，不需要保留日志信息

```shell
nohup ./cerebro >/dev/null 2>&1 &
```
启动指定端口号

```shell
nohup ./cerebro -Dhttp.port=8888 >/dev/null 2>&1 &
```



##### 2.2.5、连接

使用浏览器访问：

```url
http://192.168.150.101:8888
```



