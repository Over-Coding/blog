---
title: Ubuntu共享目录
date: 2020-03-03 19:14:03
categories:
	- Linux
tags: 
	- Linux
	- Ubuntu
toc: true
---



### 1、samba 安装
```
sudo apt-get install samba
```


### 2、创建共享目录

```
mkdir [目录]
```


### 3、修改samba配置文件

>1. 备份默认配置文件
```
sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.bak
```
>2. 编辑配置文件
```
sudo vim /etc/samba/smb.conf
```
>3. 在文件最后添加
```
[share]
    path = [共享目录]    
    available = yes      
    browsealbe = yes      
    public = yes      
    writable = yes
    comment = [共享目录描述]
```
**注意**

 >推荐为共享目录创建指定的用户，修改共享目录的权限为指定用户。