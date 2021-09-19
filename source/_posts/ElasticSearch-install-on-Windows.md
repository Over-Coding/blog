---
title: Windows下安装Elasticsearch
date: 2020-02-09 14:50:22
categories:
	- Elasticsearch
tags:
	- Elasticsearch
toc: true
typora-root-url: ElasticSearch-install-on-Windows
---

### 1、前言

因es版本原因，es5.x 以上不能按原来插件的方式安装elasticsearch-head，故需要新的方式安装使用。我这是在win10下安装测试的。



### 2、环境准备

jdk1.8 以上。



### 3、安装ElasticSearch
#### 3.1、下载

官网地址：https://www.elastic.co/cn/downloads/elasticsearch
按自己的需求下载相应的版本，我这里选择的是 5.6.16版本的zip包。
**提示：es5.x 以上版本最低需要jdk1.8。**



#### 3.2、安装

解压安装包到自己指定的位置。



#### 3.3、修改配置文件

进入到 elasticsearch-5.6.16\config 目录下，修改elasticsearch.yml文件。
去除以下注释：

```yml
集群名称（可自定义）
cluster.name: my-application

节点名称（可自定义）
node.name: node-1

数据存储路径（修改为你自己的路径）
path.data: /path/to/data

日志存储路径（修改为你自己的路径）
path.logs: /path/to/logs

监控ip
network.host: 0.0.0.0

http访问端口号
http.port: 9200
```



#### 3.4、运行

进入到 elasticsearch-5.6.16\bin 目录下，双击运行 elasticsearch.bat。



#### 3.5、测试

在浏览器上输入 http://localhost:9200 ，页面上出现节点信息，就说明安装成功。



### 4、安装node

#### 4.1、下载

官网地址：http://nodejs.cn/download/
按自己的需求下载相应的版本，我这里选择的是 版本：10.16.0 windows安装包（.msi) 64位。



#### 4.2、安装

双击安装即可。



#### 4.3、测试

在命令行中运行。

```shell
node -v
```



### 5、安装elasticsearch-head

#### 5.1、下载

地址：https://github.com/mobz/elasticsearch-head



#### 5.2、安装

解压安装包到自己指定的位置。



#### 5.3、配置

进入到 elasticsearch-head-master 目录下，修改 Gruntfile.js 文件，添加 hostname: '*',

![](1.PNG)

进入到 elasticsearch-head-master\_site 目录下，修改 app.js 文件。
将 localhost 改为安装elasticsearch的服务器ip地址，如果是本机，则不需要更改。

![](2.PNG)

进入到 elasticsearch-5.6.16\config 目录下，修改elasticsearch.yml文件。
在文件的末尾添加以下配置：

```yml
设置当前节点为主节点
node.master: true
node.data: true

开启跨域
http.cors.enabled: true 

允许所有域名访问
http.cors.allow-origin: "*"
```



#### 5.4、安装grunt-cli

在命令行中执行以下命令（比较慢）：

```shell
npm install -g grunt-cli
```



#### 5.5、初始化依赖

进入到 elasticsearch-5.6.16 目录下，在命令行中执行：

```shell
npm install
```



#### 5.6、运行

到目前为止所有的安装都已完成，以后每次启动都需要进入到 elasticsearch-5.6.16 目录下，在命令行中执行：

```shell
npm run start
```



#### 5.7、测试

在浏览器上输入 http://localhost:9100 ,出现下图，说明安装成功。

![](3.PNG)

