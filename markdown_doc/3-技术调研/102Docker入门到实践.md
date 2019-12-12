---
title: 102Docker入门到实践
tags: Docker-1
renderNumberedHeading: true
grammar_cjkRuby: true
---
对比阅读
[W3CSchool-docker入门到实践](https://www.w3cschool.cn/reqsgr/)
[docker入门到实践](https://legacy.gitbook.com/book/yeasy/docker_practice/details)

[toc]
## Docker基础
### Docker简介
#### Docker是什么？
![Docker Architecture](./images/1575428921115.png)
- go语言开发，基于linux内核技术，对线程封装隔离，属于操作系统层面的虚拟技术。
- 隔离了宿主机和其他的进程，也被成为容器
- 演进至今，基于 runC 和 containered实现
	- runc: linux命令行工具，创建和运行容器
	- containered：守护进程，在容器执行和监督下载镜像中管理容器生命周期

![传统虚拟化](./images/1575429180711.png)

![docker虚拟化技术](./images/1575429452458.png)

Docker和传统虚拟化技术区别
- 传统虚拟化技术：硬件虚拟化，运行一套完整的操作系统
- Docker：直接运行在主机内核，没有自己的内核和虚拟硬件，更轻巧
#### 为什么用Docker?
Docker和虚拟机的特性对比

|   特性  |    Docker |  虚拟机   |
| --- | --- | --- |
|   启动速度  | 秒级    |   分钟级  |
|   磁盘占用  |  MB   |   GB  |
|    性能 |  接近原生物理机   |    性能损耗大 |
|系统支持量|单机支持上千个容器   | 一般几十个  |
| CI/CD|开发、测试、生产环境一致 | 无成熟体系|
|迁移/扩展|跨平台，可复制|较为复杂|

### 基本概念
Docker由镜像、容器、仓库三个部分组成
#### 镜像
- Docker镜像是特殊的文件系统，提供容器运行需要的程序、库、资源和配置，以及配置参数，不包含动态数据，在构建后就不会发生改变。
- 分层存储：镜像是一个虚拟概念，实际由一组或者说多层文件系统联合构成。
- 镜像构建，会一层一层构建，前一层是后一层的基础。分层存储使得镜像的复制和定制更容易。
 
#### 容器
- 容器和镜像的关系，就像面向对象的‘类’和‘对象’一样。
- 容器的实质是进程。容器进程有属于自己独立的命名空间。
- 容器也是分层存储，运行以镜像为基础层，构建当前容器的存储层，成为'容器存储层'，其生命周期和容器一样。
- 容器不应向其存储层内写入任何数据，要保持无状态化。所有的文件写入操作，应使用 数据卷（Volume）、或者绑定宿主目录。数据卷的生存周期独立于容器，容器消亡，数据卷不会消亡。

#### 仓库
- 镜像集中存储和分发的服务。
- <仓库名>:<标签>对应一个镜像
- Docker Registry公开服务，开放给用户使用、允许用户管理镜像的 Registry 服务。
- 除了使用公开服务外，用户还可以在本地搭建[私有 Docker Registry](https://yeasy.gitbooks.io/docker_practice/content/repository/registry.html)。
### 安装
[ubuntu、macos安装指南](https://yeasy.gitbooks.io/docker_practice/content/install/)

mac系统最低为 macOS Sierra 10.12
- homebrew安装: **brew cask install docker**
- 手动安装：
	- [下载Docker Desktop for Mac](https://download.docker.com/mac/stable/Docker.dmg)
	- 验证：**docker --version**

镜像加速器
国内从 Docker Hub 拉取镜像有时会遇到困难，此时可以配置镜像加速器，由于镜像服务可能出现宕机，建议同时配置多个镜像。
```
使用docker info查看加速器是否生效
$docker info
Registry Mirrors: https://dockerhub.azk8s.cn/
```
- Azure 中国镜像 https://dockerhub.azk8s.cn
- 阿里云加速器(需登录账号获取)
- 七牛云加速器 https://reg-mirror.qiniu.com

![mac配置阿里云镜像加速器](./images/1575443816458.png)
	
### 基本使用
#### 镜像使用
#### 容器使用
#### 仓库使用

## DockerFile
### 基本结构
### 指令解析
### 通过DockerFile构建镜像

## Docker数据管理
管理数据的两种方式：数据卷和数据卷容器
### 数据卷
### 数据卷容器
### 数据卷备份、恢复和迁移

## Docker网络配置
### 基础网络
#### 容器联网
#### 外部访问容器


### 高级网络配置

## Docker实战
## Docker安全
## Docker底层实现
## Docker三个项目
## Docker附录