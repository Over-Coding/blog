---
title: Docker安装
date: 2020-02-02 14:09:27
categories:
	- Docker
tags: 
	- Docker
	- 安装
toc: true
---



### 1、Ubuntu 安装
#### step 1: 安装依赖工具
```
sudo apt-get update

sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
```
#### step 2: 安装GPG证书
```
curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
```
#### step 3: 添加软件源信息
```
sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
```
#### step 4: 更新并安装Docker-CE
```
sudo apt-get -y update
sudo apt-get -y install docker-ce
```
#### step 5: 配置镜像加速器（针对Docker客户端版本大于 1.10.0 的用户）
```
sudo mkdir -p /etc/docker 
sudo tee /etc/docker/daemon.json <<-'EOF' 
{ 
  "registry-mirrors":["镜像加速地址"] 
} 
EOF 
sudo systemctl daemon-reload 
sudo systemctl restart docker
```



### 2、CentOS7 安装

#### step 1: 安装依赖工具
```
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```
#### step 2: 添加软件源信息
```
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```
#### step 3: 更新并安装Docker-CE
```
sudo yum makecache fast
sudo yum -y install docker-ce
```
#### step 4: 开启Docker服务
```
sudo service docker start
```
#### step 5: 配置镜像加速器（针对Docker客户端版本大于 1.10.0 的用户）
```
sudo mkdir -p /etc/docker 
sudo tee /etc/docker/daemon.json <<-'EOF' 
{ 
  "registry-mirrors":["镜像加速地址"] 
} 
EOF 
sudo systemctl daemon-reload 
sudo systemctl restart docker
```



### 3、检验

```
docker version
```