# 概述

Docker 是一个开源的应用容器引擎，容器是完全使用沙箱机制，相互之间不会有任何接口（类似 iPhone 的 app）,更重要的是容器性能开销极低



Docker 使用客户端-服务器 (C/S) 架构模式，使用远程API来管理和创建Docker容器



- **镜像（Image）**：Docker 镜像（Image），就相当于是一个 root 文件系统。比如官方镜像 ubuntu:16.04 就包含了完整的一套 Ubuntu16.04 最小系统的 root 文件系统
- **容器（Container）**：镜像（Image）和容器（Container）的关系，就像是面向对象程序设计中的类和实例一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等
- **仓库（Repository）**：仓库可看着一个代码控制中心，用来保存镜像





# 安装

## 环境准备

### 系统要求

centos7 64位

### 移除非官方软件包

有些系统上已经安装了旧版本的 docker，新版本的 docker是 docker-engine

```shell
yum -y remove docker
```

执行该命令只会移除旧版本的 Docker， /var/lib/docker 目录中的内容不会被删除，因此，旧版本Docker所创建的镜像、容器、卷都会保留下来



### 配置yum源

```sh
vim /etc/yum.repos.d/docker.repo
#（添加以下内容）
	
	[dockerrepo]
  name=Docker Repository
  baseurl=https://yum.dockerproject.org/repo/main/centos/$releasever/
  enabled=1
  gpgcheck=1
  gpgkey=https://yum.dockerproject.org/gpg
```



### 安装Docker

```shell
	# 安装最新版本的docker
	yum -y install docker-engine
	#	在生产环境中，可能需要安装指定版本的docker，并不是总是安装最新，执行下面命令，查看docker可用版本
	yum list docker-engine.x86_64  --showduplicates |sort -r
	# 安装指定版本docker
	yum -y install docker-engine-1.13.0
	#	启动docker
	systemctl start	docker
	# 执行下面命令，验证是否安装正确
	docker run hello-world
	如果出现 unable	to find image 'hello-world:latest'
	# 查看docker版本
	docker version
```



### 卸载docker

```shell
#	卸载docker软件包
yum -y remove docker-engine
#	如果要删除镜像、容器、卷以及自定义配置文件，执行下面命令
rm -rf /var/lib/docker
```

