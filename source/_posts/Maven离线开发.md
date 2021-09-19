---
title: Maven离线开发
date: 2020-03-11 10:45:16
categories:
	- Maven
tags: 
	- Maven
toc: true
typora-root-url: Maven离线开发
---



### 1、背景
开始在有网环境下开发项目，后面因为各种原因，需要在无网环境下进行开发，因此需要在无网环境下运行Maven项目。



### 2、解决方案

#### 2.1、配置setting.xml
1. 设置本地仓库路径。

```xml
<localRepository>本地仓库路径</localRepository>
```
2. 设置镜像地址指向本地仓库路径。

```xml
<mirrors>
    <mirror>
        <id>central</id>
        <name>central</name>
        <url>file://本地仓库路径</url>
        <mirrorOf>*</mirrorOf>
    </mirror>
 </mirrors>
```



#### 2.2、准备本地仓库

在有网环境下下载好项目依赖的所有jar包，然后导入到无网环境下的本地仓库路径下。



#### 2.3、修改idea配置

1. 修改maven配置，勾选 Work offline。

![](1.PNG)

2. 修改配置文件路径为上文修改的setting.xml文件路径。

![](2.PNG)

3. 重启idea,就可以使用了。