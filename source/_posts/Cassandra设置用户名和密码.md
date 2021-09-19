---
title: Cassandra设置用户名和密码
date: 2020-02-28 08:05:59
categories:
	- Cassandra
tags: 
	- Cassandra
toc: true
---

### 1、修改cassandra.yaml
authenticator：AllowAllAuthenticator （允许所有认证登录）
修改为
authenticator: PasswordAuthenticator （允许密码认证登录）

重启服务。



### 2、使用默认用户登录

使用默认用户：cassandra，密码：cassandra 登录
进入到bin/目录下，运行命令：

```shell
./cqlsh [ip] -u cassandra -p cassandra
```



### 3、创建新用户

运行命令：

```cql
create user 用户名 with password '密码' superuser;
```
**注意：密码左右两侧需要 单引号 括起来**



### 4、使用新用户登录

运行命令：

```shell
./cqlsh [ip] -u 用户名 -p 密码
```



### 5、删除默认用户

运行命令：

```cql
drop user cassandra;
```



### 6、测试

使用默认用户登录，提示无法登录，则设置成功。
运行命令：

```shell
./cqlsh [ip] -u cassandra -p cassandra
```