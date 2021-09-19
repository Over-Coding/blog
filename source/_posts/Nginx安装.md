---
title: Nginx安装
date: 2020-03-14 15:11:13
categories:
	- Nginx
tags: 
	- Nginx
	- 安装
toc: true
typora-root-url: Nginx安装
---



#### 1、简介
Nginx (engine x) 是一个高性能的HTTP和反向代理web服务器，同时也提供了IMAP/POP3/SMTP服务。其特点是占有内存少，并发能力强，事实上nginx的并发能力在同类型的网页服务器中表现较好，中国大陆使用nginx网站用户有：百度、京东、新浪、网易、腾讯、淘宝等。(来自百度百科)



#### 2、常用的使用场景

* 正向代理

  概念：正向代理，意思是一个位于客户端和原始服务器(origin server)之间的服务器，为了从原始服务器取得内容，客户端向代理发送一个请求并指定目标(原始服务器)，然后代理向原始服务器转交请求并将获得的内容返回给客户端。客户端才能使用正向代理。

* 反向代理

  概念：反向代理服务器位于用户与目标服务器之间，但是对于用户而言，反向代理服务器就相当于目标服务器，即用户直接访问反向代理服务器就可以获得目标服务器的资源。同时，用户不需要知道目标服务器的地址，也无须在用户端作任何设定。

* 负载均衡

  概念：负载均衡，英文名称为Load Balance，其含义就是指将负载（工作任务）进行平衡、分摊到多个操作单元上进行运行，例如FTP服务器、Web服务器、企业核心应用服务器和其它主要任务服务器等，从而协同完成工作任务。

* 动静分离

  动静分离是指在web服务器架构中，将静态页面与动态页面或者静态内容接口和动态内容接口分开不同系统访问的架构设计方法，进而提升整个服务访问性能和可维护性。



#### 3、下载、安装

* 下载
>地址：http://nginx.org/en/download.html
* 安装
>1. 安装相关依赖
```shell
yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel
```
>2. 解压
```shell
tar -zxvf nginx-1.16.1.tar.gz
```
>3. 进入到解压目录，运行configure脚本
```shell
./configure
```
>4. 编译安装
```shell
make && make install
```
>5. nginx已经安装完成，安装目录为/usr/local/nginx，目录结构如下：

![](nginx目录结构.PNG)



#### 4、常用命令

* 查看帮助信息
```shell
nginx -h
```

* 查看nginx版本
```shell
nginx -v
```

* 查看nginx详细版本信息，还显示配置参数信息
```shell
nginx -V
```

* 启动nginx
```shell
nginx
```

* 指定配置文件启动nginx
```shell
nginx -c filename
```

* 优雅停止nginx，有连接时会等连接请求完成再杀死worker进程
```shell
nginx -s quit
```

* 快速停止nginx，可能并不保存相关信息
```shell
nginx -s stop
```

* 重新载入nginx，当配置信息修改后，需要重新加载配置时使用
```shell
nginx -s reload
```

* 重新打开日志文件，一般用于切割日志
```shell
nginx -s reopen
```

* 检验配置文件是否有错
```shell
nginx -t filename
```



#### 5、配置文件介绍

* 位置

  安装目录下的 conf 文件夹下的 nginx.conf 文件，在 conf 文件夹下还存放着nginx服务器的其他基础配置。

* 内容

去除注释后的文件内容如下：

```conf
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;
    
    keepalive_timeout  65;
    
    server {
        listen       80;
        server_name  localhost;
        
        location / {
            root   html;
            index  index.html index.htm;
        }
        
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```
根据上述文件内容，可以将 nginx.conf 配置文件分为三部分。

1. **全局块**

   从配置文件开始到 events 块之间的内容，主要会设置一些影响 nginx 服务器整体运行的配置指令，主要包括配置运行 nginx 服务器的用户（组）、允许生成的 worker process 数、进程 PID 存放路径、日志存放路径和类型以及配置文件的引入等。

2. **events块**

   events 块涉及的指令主要影响 nginx 服务器与用户的网络连接，常用的设置包括是否开启对多 worker process 下的网络连接进行序列化，是否允许同时接收多个网络连接，选取哪种事件驱动模型来处理连接请求，每个 worker process 可以同时支持的最大连接数等。

3. **http块**

   这块算是 nginx 服务器配置中最繁琐的部分，代理、缓存和日志定义等绝大多数功能和第三方模块的配置都在这里。
   注意：http 块包括 http 全局块、server 块。每个 http 块可以包括多个 server 块，每个 server 块就相当于一个虚拟主机。

   * **http 全局块** 
     http 全局块配置的指令包括文件引入、MIME-TYPE 定义、日志自定义、连接超时时间、单链接请求数上限等。

   * **server 块**
     server 块和虚拟主机有密切关系，虚拟主机从用户角度看，和一台独立的硬件主机是完全一样的，该技术的产生是为了节省互联网服务器硬件成本。
     注意：每个 server 块也分为全局 server 块，以及可以同时包含多个 location 块。

     * **server 全局块**
       最常见的配置是本虚拟主机的监听配置和本虚拟主机的名称或IP配置。

     * **location 块**
       location 块主要作用是基于 nginx 服务器接收到请求字符串，对虚拟主机名称之外的字符串进行匹配，对特定的请求进行处理。地址定向、数据缓存和应答控制等功能，还有许多第三方模块的配置也在这里进行。



#### 6、nginx + keepalived 高可用

##### 6.1、主从模式
TODO
##### 6.2、双主模式
TODO