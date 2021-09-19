---
title: Nginx深入原理
date: 2020-03-22 15:28:19
categories:
	- Nginx
tags: 
	- Nginx
	- 深入原理
toc: true
typora-root-url: Nginx深入原理
---



### 1、概述
![](1.PNG)

![](2.PNG)



### 2、master-workers 的机制的好处

首先，对于每个 worker 进程来说，独立的进程，不需要加锁，所以省掉了锁带来的开销，
同时在编程以及问题查找时，也会方便很多。其次，采用独立的进程，可以让互相之间不会
影响，一个进程退出后，其它进程还在工作，服务不会中断，master 进程则很快启动新的
worker 进程。当然，worker 进程的异常退出，肯定是程序有 bug 了，异常退出，会导致当
前 worker 上的所有请求失败，不过不会影响到所有请求，所以降低了风险。



### 3、worker数量设置

Nginx 同 redis 类似都采用了 io 多路复用机制，每个 worker 都是一个独立的进程，但每个进
程里只有一个主线程，通过异步非阻塞的方式来处理请求， 即使是千上万个请求也不在话下。每个 worker 的线程可以把一个 cpu 的性能发挥到极致。所以 worker 数和服务器的 cpu数相等是最为适宜的。设少了会浪费 cpu，设多了会造成 cpu 频繁切换上下文带来的损耗。

>worker_processes 4
>#work 绑定 cpu(4 work 绑定 4cpu)。
>worker_cpu_affinity 0001 0010 0100 1000
>#work 绑定 cpu (4 work 绑定 8cpu 中的 4 个) 。
>worker_cpu_affinity 0000001 00000010 00000100 00001000



### 4、worker_connection 数量设置

这个值是表示每个 worker 进程所能建立连接的最大值，所以，一个 nginx 能建立的最大连接数，应该是 worker_connections * worker_processes。当然，这里说的是最大连接数，对于HTTP 请 求 本 地 资 源 来 说 ， 能 够 支 持 的 最 大 并 发 数 量 是 worker_connections * worker_processes，如果是支持 http1.1 的浏览器每次访问要占两个连接，所以普通静态访问最大并发数是：worker_connections * worker_processes /2，而如果是 HTTP 作 为反向代理来说，最大并发数量是：worker_connections * worker_processes/4。因为作为反向代理服务器，每个并发会建立与客户端的连接和与后端服务的连接，会占用两个连接。