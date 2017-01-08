---
layout: post
notes: true
subtitle: Docker-从入门到实践
comments: false
author: "yeasy@github"
date: 2017-01-07 00:00:00

---

![](/img/notes/operation/dockerPrimer/docker_primer.png)

*   目录
{:toc }

# 前言

Docker是个划时代的开源项目，它彻底释放了计算虚拟化的威力，极大提高了应用的运行效率，降低了云计算资源供应的成本！

# Docker简介

## 什么是Docker

Docker在容器的基础上，进行了进一步的封装，从文件系统、网络互联到进程隔离等等，极大的简化了容器的创建和维护。

Docker和传统虚拟化方式的不同：

*	传统虚拟机技术是虚拟出一套硬件后，在其上运行一个完整操作系统，在该系统上再运行所需应用进程
*	容器内的应用进程直接运行于宿主的内核，容器内没有自己的内核，而且也没有进行硬件虚拟。因此容器要比传统虚拟机更为轻便。

![](/img/notes/operation/dockerPrimer/virtualization.png)

![](/img/notes/operation/dockerPrimer/docker.png)

## 为什么要用Docker

*	更高效的利用系统资源
*	更快速的启动时间
*	一致的运行环境
*	持续交付和部署
*	更轻松的迁移
*	更轻松的维护和扩展

| 特性 | 容器 | 虚拟机 |
| ---- | ---- | ------ |
| 启动 | 秒级 | 分钟级 |
| 硬盘使用 | 一般为MB | 一般为GB |
| 性能 | 接近原生 | 弱于 |
| 系统支持量 | 单机支持上千个容器 | 一般几十个 |

# 基本概念

三个基本概念：

*	镜像(Image)
*	容器(Container)
*	仓库(Repository)

## 镜像

### Docker镜像

Docker镜像(Image)就相当于是一个root文件系统。

Docker镜像是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。镜像不包含任何动态数据，其内容在构建之后也不会被改变。

### 分层存储

在Docker设计时，充分利用Union FS的技术，将其设计为分层存储的架构。镜像只是一个虚拟的概念，其实际体现并非由一个文件组成，而是由一组文件系统组成，或者说，由多层文件系统联合组成。

镜像构建时，会一层层构建，前一层是后一层的基础。每一层构建完就不会再发生改变，后一层上的任何改变只发生在自己这一层。因此，在构建镜像的时候，需要额外小心，每一层尽量只包含该层需要添加的东西，任何额外的东西应该在该层构建结束前清理掉。

分层存储的特征还使得镜像的复用、定制变得更为容易。甚至可以用之前构建好的镜像作为基础层，然后进一步添加新的层，以定制自己所需的内容，构建新的镜像。

## 容器

镜像(Image)和容器(Container)的关系，就像是面向对象程序设计中的*类*和*实例*一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。

容器的实质是进程，但与直接在宿主执行的进程不同，容器进程运行于属于自己的独立的命名空间。因此容器可以拥有自己的root文件系统、自己的网络配置、自己的进程空间，甚至自己的用户ID空间。容器内的进程是运行在一个隔离的环境里，使用起来就好像是在一个独立于宿主的系统下操作一样。这种特性使得容器封装的应用比直接在宿主运行更加安全。

每一个容器运行时，是以镜像为基础层，在其上创建一个当前容器的存储层，我们可以称这个为容器运行时读写而准备的存储层为容器存储层。

容器存储层的生命周期和容器一样，容器消亡时，容器存储层也随之消亡。

按照Docker最佳实践的要求，容器不应该向其存储层写入任何数据，容器存储层要保持无状态化。所有的文件写入操作，都应该使用数据卷(Volume)、或者绑定宿主目录，在这些位置的读写会跳过容器存储层，直接对宿主（或网络存储）发生读写，其性能和稳定性更高。

数据卷的生命周期独立于容器，容器消亡，数据卷不会消亡。因此，使用数据卷，容器可以随意删除、重新run，数据却不会丢失。

## 仓库

### Docker Registry

集中的存储、分发镜像的服务。

一个Docker Registry中可以包含多个仓库(Registory)；每个仓库可以包含多个标签(Tag)；每个标签对应一个镜像。

### Docker Registry公开服务

Docker Registry公开服务是开放给用户使用、允许用户管理镜像的Registry服务。一般这类公开服务允许用户免费上传、下载公开的镜像，并可能提供收费服务供用户管理私有镜像。

最常用的Registry公开服务是官方的Docker Hub，这也是默认的Registry，并拥有大量的高质量的官方镜像。

### 私有Docker Registry

用户可以在本地搭建私有Docker Registry。

# 安装

## Ubuntu、Debian

### 系统要求

Ubuntu发行版中，在生产环境中推荐使用LTS版本。

Docker需要安装在64位的x86平台或ARM平台上（如树莓派），并且要求内核版本不低于3.10。

可以通过如下命令检查自己的内核版本详细信息：

	$ uname -a

### 使用脚本自动安装

	curl -sSL https://get.docker.com/ | sh
	
### 手动安装

	$ sudo apt-get install linux-image-extra-$(uname -r) linux-image-extra-virtual
	
#### 添加APT镜像源

	$ sudo apt-get update
	$ sudo apt-get install apt-transport-https ca-certificates

为了确认所下载软件包的合法性，需要添加 Docker 官方软件源的 GPG 密钥。

	$ sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
	
用下面的命令将APT源添加到source.list：

	$ echo "<REPO>" | sudo tee /etc/apt/sources.list.d/docker.list
	
添加成功后，更新 apt 软件包缓存。

	$ sudo apt-get update
	
#### 安装Docker

	$ sudo apt-get install docker-engine
	
### 启动Docker引擎

Ubuntu 12.04/14.04、Debian 7 Wheezy：

	$ sudo service docker start
	
Ubuntu 16.04、Debian 8 Jessie/Stretch：

	$ sudo systemctl enable docker
	$ sudo systemctl start docker
	
### 建立docker用户组

默认情况下， docker 命令会使用 Unix socket 与 Docker 引擎通讯。而只有root 用户和 docker 组的用户才可以访问 Docker 引擎的 Unix socket。出于安全考虑，一般 Linux 系统上不会直接使用 root 用户。因此，更好地做法是将需要使用 docker 的用户加入 docker 用户组。

建立docker组：

	$ sudo groupadd docker

将当前用户加入docker组：

	$ sudo usermod -aG docker $USER

### 参考文档

*	[Docker官方Ubuntu安装文档](https://docs.docker.com/engine/installation/linux/ubuntulinux/)
*	[Docker官方Debian安装文档](https://docs.docker.com/engine/installation/linux/debian/)