---
title: Nginx反向代理
date: 2020-03-21 14:36:01
categories:
	- Nginx
tags: 
	- Nginx
toc: true
typora-root-url: Nginx反向代理
---



### 1、案例一
#### 1.1、需求
使用 nginx （192.168.150.101）反向代理，在本机（192.168.150.100）访问 www.ld.com 直接跳转到 192.168.150.102:8080 。



#### 1.2、配置

* step1：修改本机 hosts 文件，位置：C:\Windows\System32\drivers\etc\，并在文件最后添加如下配置。
```file
192.168.150.101 www.ld.com
```
* step2：修改nginx.conf，在文件中的 http 块中添加如下配置。
```conf
server{
    listen  80;
    server_name www.ld.com
    
    location / {
        proxy_pass   http://192.168.150.102:8080;
    }
}
```
* step3：启动tomcat，使用默认的端口号。
* step4：在浏览器上访问 www.ld.com，出现如下效果就算成功。
![](1.PNG)

### 2、案例二

#### 2.1、需求
使用 nginx（192.168.150.101）反向代理，在本机（192.168.150.100）访问 不同的地址，跳转到不同的tomcat中。
地址1：www.ld.com/102/index.html ,目标：192.168.150.102:8080
地址2：www.ld.com/103/index.html ,目标：192.168.150.103:8080



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
    
    location ~ /102/ {
        proxy_pass   http://192.168.150.102:8080;
    }
    
    location ~ /103/ {
        proxy_pass   http://192.168.150.103:8080;
    }
}
```
* step3：在102和103服务器上 tomcat 的webapp目录下分别添加文件夹102,103,并在文件夹下添加index.html，并启动。
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
* step4：在浏览器上分别访问不同的url

  地址1：www.ld.com/102/index.html 

![](2.1.PNG)

​		地址2：www.ld.com/103/index.html 

![](2.2.PNG)