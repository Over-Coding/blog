---
title: Nginx负载均衡
date: 2020-03-21 16:41:57
categories:
	- Nginx
tags: 
	- Nginx
toc: true
typora-root-url: Nginx负载均衡
---



### 1、案例
#### 1.1、需求
使用 nginx （192.168.150.101）负载均衡，在本机（192.168.150.100）访问 www.ld.com/test/index.html 跳转到 192.168.150.102:8080/test/index.html或192.168.150.103:8080/test/index.html 。



#### 1.2、配置

* step1：修改本机 hosts 文件，位置：C:\Windows\System32\drivers\etc\，并在文件最后添加如下配置。
```file
192.168.150.101 www.ld.com
```
* step2：修改nginx.conf，在文件中的 http 块中添加如下配置。
```conf
upstream ldserver {
     server 192.168.150.102:8080;
     server 192.168.150.103:8080;
}

server{
    listen  80;
    server_name www.ld.com
    
    location / {
        proxy_pass   http://ldserver;
    }
}
```
* step3：在102和103服务器上 tomcat 的webapp目录下分别添加文件夹test,并在文件夹下添加index.html，并启动。
>102服务器上的index.html
```html
<!DOCTYPE html>
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
	<title></title>
</head>
<body>
	<h1>你好，我是102</h1>
</body>
</html>
```
>103服务器上的index.html
```html
<!DOCTYPE html>
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
	<title></title>
</head>
<body>
	<h1>你好，我是103</h1>
</body>
</html>
```
* step4：在浏览器上多次访问 http://www.ld.com/test/index.html，循环显示如下页面：

![](1.PNG)

![](2.PNG)



### 2、Nginx 支持的负载均衡策略

* 轮询（默认）

每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。

* weight

weight 代表权重，默认为1，权重越高被分配的客户端越多
例子：

```conf
upstream server_pool {
    server  192.168.150.102:8080 weight=10;
    server  192.168.150.103:8080 weight=5;
}
```
* ip_hash

每个请求按访问 ip 的 hash 结果分配，这样每个访客固定访问一个后端服务器，可以解决session 的问题。
例子：

```conf
upstream server_pool {
    ip_hash;
    server  192.168.150.102:8080;
    server  192.168.150.103:8080;
}
```
* fair

按后端服务器的响应时间来分配请求，响应时间短的优先分配。
**注意：需要安装插件，因为是第三方的。**
例子：

```conf
upstream server_pool {
    fair;
    server  192.168.150.102:8080;
    server  192.168.150.103:8080;
}
```