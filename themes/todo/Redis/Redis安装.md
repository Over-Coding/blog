### 1、下载

地址：https://redis.io/download



### 2、安装

解压：

```sh
tar -zxvf redis-6.2.5.tar.gz
```

编译安装：

**注意：需要安装gcc环境，默认是安装在/usr/local/bin**

```sh
cd redis-6.2.5
#编译
make

#安装
#make install

#或者安装到指定目录下
make install PREFIX=/opt/module/redis
```

Tips：

如果未安装gcc，先安装gcc

```sh
#检查是否安装了gcc
gcc -v

#安装
sudo yum install gcc
```



### 3、修改配置

复制配置文件

```sh
cp /opt/software/redis-6.2.5/redis.conf /opt/module/redis/conf
```

编辑配置文件

```sh
cd /opt/module/redis/conf

vim redis.conf
```

修改如下：

```conf
#绑定ip，即指定客户端ip，可以绑定多个，或者如下，表示所有ip都可以访问
bind 0.0.0.0
#指定端口号
port 6379
#指定后台启动
daemonize yes
#指定日志位置，日志文件名称
logfile "/opt/module/redis/logs/redis.log"
#指定数据存储位置
dir /opt/module/redis/data
```



### 4、启动

```sh
/opt/module/redis/bin/redis-server /opt/module/redis/conf/redis.conf
```

