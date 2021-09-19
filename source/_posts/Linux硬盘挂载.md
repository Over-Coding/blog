---
title: Linux硬盘挂载
date: 2020-03-01 09:21:19
categories:
	- Linux
tags: 
	- Linux
toc: true
---



### 1、查看硬盘
```shell
lsblk
```



### 2、分区

```shell
#使用parted来对GPT磁盘操作，进入交互式模式
sudo parted /dev/[设备名] 
#将磁盘格式化为GPT
(parted) mklabel gpt
#将所有容量分为一个主分区
(parted) mkpart primary ext4 0% 100%
#打印当前分区
(parted) p
#退出
(parted) q
```



### 3、格式化

```shell
sudo mkfs.ext4 /dev/[分区名]
```



### 4、挂载

```shell
#在根目录创建一个文件夹
sudo mkdir /data
#将磁盘分区挂载到文件夹下
sudo mount /dev/[分区名] /data
```



### 5、查看UUID

```shell
sudo blkid
```



### 6、修改/etc/fstab

>在fstab文件的最后添加一行
```file
UUID=cf116c95-b7f0-4ce4-b0da-7f2856784cc3   /data   ext4    defaults    0 2
```