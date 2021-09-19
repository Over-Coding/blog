---
title: SpringBoot引入本地jar包
date: 2020-01-03 15:09:27
categories:
	- SpringBoot
tags: 
	- SpringBoot
toc: true
---



### 1、添加本地依赖jar包
在 dependencies 中添加如下配置：

```xml
<dependency>    
   <groupId>weather-log</groupId>    
   <artifactId>weather-log</artifactId>    
   <version>0.1</version>    
   <scope>system</scope>    
   <systemPath>${project.basedir}/src/main/resources/lib/weather-log.jar</systemPath></dependency>
```
**注意：${project.basedir}/src/main/resources/lib/weather-log.jar 一定要和你的jar包路径一样，我的是放在resources/lib文件夹下**



### 2、配置plugin

在 plugins 中添加如下配置：

```xml
<plugin>    
    <groupId>org.springframework.boot</groupId>    
    <artifactId>spring-boot-maven-plugin</artifactId>    
    <configuration>        
        <includeSystemScope>true</includeSystemScope>        
        <fork>true</fork>    
    </configuration>
</plugin>
```

就是这么简单，完成了，真的没了。。。