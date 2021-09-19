---
title: Nginx动静分离
date: 2020-03-15 09:46:08
categories:
	- Nginx
tags: 
	- Nginx
toc: true
typora-root-url: Nginx动静分离
---



### 1、概述

Nginx 动静分离简单来说就是把动态跟静态请求分开，不能理解成只是单纯的把动态页面和静态页面物理分离。严格意义上说应该是动态请求跟静态请求分开，可以理解成使用 Nginx处理静态页面，Tomcat 处理动态页面。动静分离从目前实现角度来讲大致分为两种：
* 纯粹把静态文件独立成单独的域名，放在独立的服务器上，也是目前主流推崇的方案；
* 把动态跟静态文件混合在一起发布，通过 nginx 来分开。通过 location 指定不同的后缀名实现不同的请求转发。通过 expires 参数设置，可以使浏览器缓存过期时间，减少与服务器之前的请求和流量。具体 Expires 定义：是给一个资源设定一个过期时间，也就是说无需去服务端验证，直接通过浏览器自身确认是否过期即可，所以不会产生额外的流量。此种方法非常适合不经常变动的资源。（如果经常更新的文件，不建议使用 Expires 来缓存），我这里设置 3d，表示在这 3 天之内访问这个 URL，发送一个请求，比对服务器该文件最后更新时间没有变化，则不会从服务器抓取，返回状态码304，如果有修改，则直接从服务器重新下载，返回状态码 200。



### 2、案例

#### 2.1、需求
使用 nginx（192.168.150.101）动态分离，在本机（192.168.150.100）访问不同的地址，获取192.168.150.101服务器上不同的静态资源。
地址1：www.ld.com/html/index.html
地址2：www.ld.com/image/test.png



#### 2.2、配置

* step1：修改本机 hosts 文件，位置：C:\Windows\System32\drivers\etc\，并在文件最后添加如下配置。
```file
192.168.150.101 www.ld.com
```
* step2：修改nginx.conf，在文件中的 http 块中添加如下配置。
```conf
server{
    listen  80;
    server_name www.ld.com
    #动态资源
    location /html/ {
        root    /data/;
        index index.html index.htm;
    }
    
    location /image/ {
        root    /data/;
        #打开目录浏览功能
        autoindex on;
    }
}
```
* step3：在192.168.150.101服务器根目录下创建data文件夹。

  在data文件夹下创建html文件夹，在html文件夹下添加index.html。index.html内容如下：

```html
<!DOCTYPE html>
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
	<title></title>
</head>
<body>
	<h1>你好，我是一个html</h1>
</body>
</html>
```
​	在data文件夹下创建image文件夹，在image文件夹下添加test.png。

![](test.png)

* step4：在浏览器上访问不同路径显示不同信息，测试如下：

  地址1：www.ld.com/html/index.html

![](test1.PNG)

​       地址2：www.ld.com/image/test.png

![](test2.PNG)

​		地址3（浏览目录）：www.ld.com/image/

![](test3.PNG)