---
title: spring-framework-5.1.x源码环境构建
date: 2020-06-23 11:32:18
categories:
	- spring-framework
tags: 
	- spring-framework
	- 源码
toc: true
typora-root-url: spring-framework-5.1.x源码环境构建
---



### 1、概述



### 2、配置Java环境

相信大家对配置Java环境都很熟练了，这里就略过了。

**注意：jdk版本最好是1.8以上。**



### 3、配置gradle环境

#### 3.1、下载

首先查看spring-framework使用的哪个版本的gradle构建的。这里我要构建spring-framework-5.1.x源码环境，就到github上查看相应分支的gradle配置。

![](gradle版本.PNG)

可以看到使用的是gradle-4.10.3-bin版本，下面就去官网下载对应的版本。

下载地址：https://gradle.org/releases/

#### 3.2、安装

将下载的安装包解压到安装目录下，例如：D:\tools\gradle-4.10.3



#### 3.3、配置环境变量

需要将安装地址配置到环境变量中，步骤如下：

**step1：**新建环境变量 GRADLE_HOME。

![](配置gradle环境变量.PNG)



**step2：**在Path环境变量中新增 %GRADLE_HOME%\bin。

![](配置Path环境变量.PNG)



**step3：**打开命令行工具，执行 gradle -v ，查看是否安装配置成功。

![](gradle安装结果.PNG)



#### 3.4、配置本地仓库地址

确定本地仓库地址，即gradle下载的jar存放地址。例如：E:\gradle\repository。

新建环境变量，配置 GRADLE_USER_HOME。

![](gradle本地仓库地址.PNG)



#### 3.5、配置国内镜像仓库

因为gradle默认使用的中央仓库在国外，网速比较慢，所以要配置国内的镜像仓库。

在Gradle安装目录下的 init.d 文件夹下，新建一个 init.gradle 文件，里面填写以下配置。

```gradle
allprojects {
    repositories {
        maven { url 'file:///E:/maven/repository'}
        mavenLocal()
        maven { name "Alibaba" ; url "https://maven.aliyun.com/repository/public" }
        maven { name "Bstek" ; url "http://nexus.bsdn.org/content/groups/public/" }
        mavenCentral()
    }

    buildscript { 
        repositories { 
            maven { name "Alibaba" ; url 'https://maven.aliyun.com/repository/public' }
            maven { name "Bstek" ; url 'http://nexus.bsdn.org/content/groups/public/' }
            maven { name "M2" ; url 'https://plugins.gradle.org/m2/' }
        }
    }
}
```



### 4、配置spring-framework环境

#### 4.1、下载spring-framework源码

spring源码都在github上，但是从github上下载源码网速太慢，所以我就将源码同步到了gitee上，直接从gitee上下载源码即可。在命令行中执行如下命令进行下载：

```shell
git clone -b 5.1.x git@gitee.com:lingfeng1024/spring-framework.git
```



#### 4.2、配置idea中gradle环境

源码下载好了，需要通过idea打开源码，不过，构建源码环境之前，还要配置一下idea的gradle环境。





#### 4.3、使用idea导入源码

