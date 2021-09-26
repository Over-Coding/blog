---
title: Hexo博客搭建
date: 2020-01-02 00:02:15
categories:
	- Hexo
tags: 
	- Hexo
toc: true
---

### 1、准备环境

#### 1.1、安装Git

下载地址：https://git-scm.com/download/win

安装好后，查看版本，验证是否安装成功。

```shell
git --version
```



#### 1.2、安装nodejs、npm

下载地址：https://nodejs.org/en/download/

因为nodejs包含npm,所以不需要单独安装，安装好后用以下命令来查看版本，验证是否安装成功。

```shell
node -v
npm -v
```



#### 1.3、安装cnpm

因为npm服务器不在国内，访问速度有些慢，但是淘宝团队提供了国内镜像，所以需要安装cnpm。

```shell
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

查看版本，验证是否安装成功。

```shell
cnpm -v
```



### 2、安装Hexo

#### 2.1、安装

```shell
cnpm install -g hexo-cli
```

查看版本，验证是否安装成功。

```shell
hexo -v
```



#### 2.2、初始化

先创建一个文件夹，用于存放hexo初始化的文件，我创建的文件夹的位置为E:\blog。然后在此文件夹下执行如下命令：

```shell
hexo init blog
```

其中hexo-blog为自定义的文件夹名。如果缺少module，就进入到hexo-blog文件夹，执行如下命令：

```shell
cnpm install
```

文件夹目录如下：

```txt
node_modules：npm依赖
scaffolds：生成文章的一些模板
source：存放文章
themes：主题
_config.yml：主要配置文件
package.json
package-lock.json
```



#### 2.3、运行

生成服务：

```shell
hexo g
```

运行服务：

```shell
hexo server
```

使用浏览器访问 http://localhost:4000 就可以看到生成的博客。



### 3、将Hexo部署到Gitee

#### 3.1、在gitee上创建一个仓库



#### 3.2、启动Gitee Pages服务

点击仓库服务中的Gitee Pages，然后点击开启即可。



#### 3.3、修改本地_config.yml配置文件

修改如下配置：

```yml
url: 开启的Gitee Pages服务的url
root: 仓库名称

deploy:
  type: git
  repo: 你自己的仓库地址
  branch: master
```



#### 3.4、安装deploy-git

安装deploy-git，用于部署服务。执行如下命令：

```shell
cnpm install hexo-deployer-git --save
```



#### 3.5、清空、生成和部署

```shell
hexo clean
hexo generate
hexo deploy
```



#### 3.6、刷新服务

因为gitee不会自动部署刷新，所以需要每次部署完进行手动刷新。然后就可以在浏览器访问Gitee Pages服务生成的网站地址了，此地址就是你对外的博客地址。



### 4、配置个人域名

在Gitee 上是花钱的，这个就看自己选择吧。



### 5、多终端工作

为了支持多个终端进行编写部署，使用git分支特性来实现。

#### 5.1、创建分支

#### 5.2、克隆分支到本地

```shell
git clone -b 分支名 仓库地址
```



#### 5.3、复制hexo资源

首先删除除.git文件的其他文件，然后将原先的除.deploy_git文件的其他资源文件拷贝到这里。



#### 5.4、提交

```shell
git add .
git commit -m "add source"
git push
```

注意：一般只提交以下文件

```txt
scaffolds：生成文章的一些模板
source：存放文章
themes：主题
.gitignore：过滤配置
_config.yml：主要配置文件
package-lock.json
package.json
```



#### 5.5、切换到其他电脑

1. 首先配置好环境，按照上面的步骤配置
2. 克隆分支仓库到本地
3. 执行 cnpm install 命令进行初始化
4. 编写自己的新内容
5. 然后生成、部署
6. 最后别忘了提交自己的新内容

注意：如果当前电脑已经有了上述环境，直接执行如下命令同步一下仓库就可以了。

```shell
git pull
```



### 6、修改主题

如果不喜欢hexo默认提供的主题，可以自己寻找自己喜欢的主题进行切换。

#### 6.1、下载主题

```shell
git clone 主题仓库地址 themes/主题名称
```

其中 themes/主题名称 是指将主题下载到指定位置。

这里有两个我比较喜欢的主题，我使用的是pure：

https://github.com/cofess/hexo-theme-pure

https://github.com/lxqxsyu/hexo-theme-icarus



#### 6.2、修改配置

这里需要说明一下，在根目录下有一个配置文件_config.yml，这个是用于配置hexo的，进入到 themes/pure目录中，同样有一个配置文件 _config.yml ,这个是用于配置主题的。

这里我们需要修改的是hexo配置文件，修改如下：

```yml
theme: 主题名称
```

然后重新清空、生成和部署即可。



#### 6.3、配置主题

按照自己选定的主题，进行自主配置。因为每个主题配置都不尽相同，所以这里就不做描述，可以根据主题相关文档进行配置。

### 7、配置Hexo

请参考[官方文档](https://hexo.io/zh-cn/docs/configuration)进行配置。



### 8、添加插件

