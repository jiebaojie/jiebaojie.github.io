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
	
#### 启动Docker引擎

Ubuntu 12.04/14.04、Debian 7 Wheezy：

	$ sudo service docker start
	
Ubuntu 16.04、Debian 8 Jessie/Stretch：

	$ sudo systemctl enable docker
	$ sudo systemctl start docker
	
#### 建立docker用户组

默认情况下， docker 命令会使用 Unix socket 与 Docker 引擎通讯。而只有root 用户和 docker 组的用户才可以访问 Docker 引擎的 Unix socket。出于安全考虑，一般 Linux 系统上不会直接使用 root 用户。因此，更好地做法是将需要使用 docker 的用户加入 docker 用户组。

建立docker组：

	$ sudo groupadd docker

将当前用户加入docker组：

	$ sudo usermod -aG docker $USER

### 参考文档

*	[Docker官方Ubuntu安装文档](https://docs.docker.com/engine/installation/linux/ubuntulinux/)
*	[Docker官方Debian安装文档](https://docs.docker.com/engine/installation/linux/debian/)

## CentOS

### 系统要求

Docker 最低支持 CentOS 7。

Docker 需要安装在 64 位的平台，并且内核版本不低于 3.10

### 使用脚本自动安装

	curl -sSL https://get.docker.com/ | sh

### 手动安装

#### 添加内核参数

	$ sudo tee -a /etc/sysctl.conf <<-EOF
	net.bridge.bridge-nf-call-ip6tables = 1
	net.bridge.bridge-nf-call-iptables = 1
	EOF

重新加载sysctl.conf：
	
	$ sudo sysctl -p

#### 添加yum源

	$ sudo tee /etc/yum.repos.d/docker.repo <<-'EOF'
	[dockerrepo]
	name=Docker Repository
	baseurl=https://yum.dockerproject.org/repo/main/centos/7/
	enabled=1
	gpgcheck=1
	gpgkey=https://yum.dockerproject.org/gpg
	EOF

#### 安装Docker

	$ sudo yum update
	$ sudo yum install docker-engine

#### 启动Docker引擎

	$ sudo systemctl enable docker
	$ sudo systemctl start docker

#### 建立docker用户组

	$ sudo groupadd docker
	$ sudo usermod -aG docker $USER

### 参考文档

*	[Docker 官方 CentOS 安装文档](https://docs.docker.com/engine/installation/linux/centos/)

## macOS

### 系统要求

Docker for Mac 要求系统最低为 macOS 10.10.3 Yosemite，或者 2010 年以后的 Mac 机型，准确说是带 Intel MMU 虚拟化的，最低 4GB 内存。

### 安装

#### 使用Homebrew安装

	brew cask install docker

#### 手动下载安装

下载链接：https://download.docker.com/mac/stable/Docker.dmg

配置加速器

启动终端后，通过命令可以检查安装后的 Docker 版本。

	$ docker --version
	$ docker-compose --version
	$ docker-machine --version

如果 docker version、docker info 都正常的话，可以运行一个 Nginx 服务器：

	$ docker run -d -p 80:80 --name webserver nginx

服务运行后，可以访问 http://localhost，如果看到了 "Welcome to nginx!"，就说明 Docker for Mac 安装成功了。

要停止 Nginx 服务器并删除执行下面的命令：

	$ docker stop webserver
	$ docker rm webserver
	
## 镜像加速器

*	阿里云加速器
*	DaoCloud加速器
*	灵雀云加速器

检查加速器是否生效

	$ sudo ps -ef | grep dockerd
	
如果从结果中看到了配置的 --registry-mirror参数说明配置成功。

# 镜像

镜像是Docker的三大组件之一。

Docker运行容器前需要本地存在对应的镜像，如果镜像不存在本地，Docker慧聪镜像仓库下载。

## 获取镜像

### 获取镜像

	$ docker pull ubuntu:14.04
	
### 运行

	$ docker run -it --rm ubuntu:14.04 bash
	root@e7009c6ce357:/# cat /etc/os-release
		NAME="Ubuntu"
		VERSION="14.04.5 LTS, Trusty Tahr"
		ID=ubuntu
		ID_LIKE=debian
		PRETTY_NAME="Ubuntu 14.04.5 LTS"
		VERSION_ID="14.04"
		HOME_URL="http://www.ubuntu.com/"
		SUPPORT_URL="http://help.ubuntu.com/"
		BUG_REPORT_URL="http://bugs.launchpad.net/ubuntu/"
	root@e7009c6ce357:/# exit
	exit
	$
	
*	-it：这是两个参数，一个是 -i ：交互式操作，一个是 -t 终端。我们这里打算进入 bash 执行一些命令并查看返回结果，因此我们需要交互式终获取镜像终端。
*	--rm：这个参数是说容器退出后随之将其删除。默认情况下，为了排障需求，退出的容器并不会立即删除，除非手动 docker rm 。我们这里只是随便执行个命令，看看结果，不需要排障和保留结果，因此使用 --rm 可以避免浪费空间。
*	ubuntu:14.04：这是指用 ubuntu:14.04 镜像为基础来启动容器。
*	bash：放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 bash 。

进入容器后，我们可以在 Shell 下操作，执行任何所需的命令。这里，我们执行了cat /etc/os-release ，这是 Linux 常用的查看当前系统版本的命令，从返回的结果可以看到容器内是 Ubuntu 14.04.5 LTS 系统。

最后我们通过 exit 退出了这个容器。

## 列出镜像

### 列出镜像

	$ docker images

### 镜像体积

Docker Hub 中显示的体积是压缩后的体积。在镜像下载和上传过程中镜像是保持着压缩状态的，因此 Docker Hub 所显示的大小是网络传输中更关心的流量大小。而 docker images 显示的是镜像下载到本地后，展开的大小，准确说，是展开后的各层所占空间的总和，因为镜像到本地后，查看空间的时候，更关心的是本地磁盘空间占用的大小。

由于 Docker 镜像是多层存储结构，并且可以继承、复用，因此不同镜像可能会因为使用相同的基础镜像，从而拥有共同的层。由于 Docker 使用 Union FS，相同的层只需要保存一份即可，因此实际镜像硬盘占用空间很可能要比这个列表镜像大小的总和要小的多。

### 虚悬镜像

一个特殊的镜像，这个镜像既没有仓库名，也没有标签，均为 <none> 。

由于新旧镜像同名，旧镜像名称被取消，从而出现仓库名、标签均为 <none> 的镜像。这类无标签镜像也被称为 虚悬镜像(dangling image) ，可以用下面的命令专门显示这类镜像：

	$ docker images -f dangling=true
	
一般来说，虚悬镜像已经失去了存在的价值，是可以随意删除的，可以用下面的命令删除。

	$ docker rmi $(docker images -q -f dangling=true)
	
### 中间层镜像

为了加速镜像构建、重复利用资源，Docker 会利用 中间层镜像。所以在使用一段时间后，可能会看到一些依赖的中间层镜像。默认的 docker images 列表中只会显示顶层镜像，如果希望显示包括中间层镜像在内的所有镜像的话，需要加 -a 参数。

	$ docker images -a
	
这样会看到很多无标签的镜像，与之前的虚悬镜像不同，这些无标签的镜像很多都是中间层镜像，是其它镜像所依赖的镜像。这些无标签镜像不应该删除，否则会导致上层镜像因为依赖丢失而出错。实际上，这些镜像也没必要删除，因为相同的层只会存一遍，而这些镜像是别的镜像的依赖，因此并不会因为它们被列出来而多存了一份，无论如何你也会需要它们。只要删除那些依赖它们的镜像后，这些依赖的中间层镜像也会被连带删除。

### 列出部分镜像

	$ docker images ubuntu
	$ docker images ubuntu:16.04
	$ docker images -f since=mongo:3.2
	$ docker images -f before=mongo:3.2
	$ docker images -f lable=com.example.version=0.1
	
### 以特定格式显示

	$ docker images -q
	5f515359c7f8
	05a60462f8ba
	fe9198c04d62
	00285df0df87
	f753707788c5
	f753707788c5
	1e0c3dd64ccd
	
--filter 配合 -q 产生出指定范围的 ID 列表，然后送给另一个 docker 命令作为参数

下面的命令会直接列出镜像结果，并且只包含镜像ID和仓库名：

	$ docker images --format "{{.ID}}: {{.Repository}}"
	
自己定义列：

	$ docker images --format "table {{.ID}}\t{{.Repository}}\t{{.Tag}}"
	
## 利用commit理解镜像构成

### 利用commit理解镜像构成

镜像是容器的基础，每次执行 docker run 的时候都会指定哪个镜像作为容器运行的基础。

	$ docker run --name webserver -d -p 80:80 nginx
	
这条命令会用 nginx 镜像启动一个容器，命名为 webserver ，并且映射了 80 端口，这样我们可以用浏览器去访问这个 nginx 服务器。

现在，假设我们非常不喜欢这个欢迎页面，我们希望改成欢迎 Docker 的文字，我们可以使用 docker exec 命令进入容器，修改其内容。

	$ docker exec -it webserver bash
	root@3729b97e8226:/# echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
	root@3729b97e8226:/# exit
	exit

我们修改了容器的文件，也就是改动了容器的存储层。我们可以通过 docker diff 命令看到具体的改动。

	$ docker diff webserver

Docker 提供了一个 docker commit 命令，可以将容器的存储层保存下来成为镜像。换句话说，就是在原有镜像的基础上，再叠加上容器的存储层，并构成新的镜像。以后我们运行这个新镜像的时候，就会拥有原有容器最后的文件变化。

	$ docker commit \
		--author "Tao Wang <twang2218@gmail.com>" \
		--message "修改了默认网页" \
		webserver \
		nginx:v2
	sha256:07e33465974800ce65751acc279adc6ed2dc5ed4e0838f8b86f0c87aa1795214

其中 --author 是指定修改的作者，而 --message 则是记录本次修改的内容。

我们可以在 docker images 中看到这个新定制的镜像：

	$ docker images nginx
	
可以用 docker history 具体查看镜像内的历史记录：

	$ docker history nginx:v2
	
新的镜像定制好后，我们可以来运行这个镜像。

	$ docker run --name web2 -d -p 81:80 nginx:v2

这里我们命名为新的服务为 web2 ，并且映射到 81 端口。

### 慎用docker commit

使用 docker commit 命令虽然可以比较直观的帮助理解镜像分层存储的概念，但是实际环境中并不会这样使用。

docker commit 命令除了学习之外，还有一些特殊的应用场合，比如被入侵后保存现场等。但是，不要使用 docker commit 定制镜像，定制行为应该使用 Dockerfile 来完成。

## 使用Dockerfile定制镜像

### 使用Dockerfile定制镜像

镜像的定制实际上就是定制每一层所添加的配置、文件。如果我们可以把每一层修改、安装、构建、操作的命令都写入一个脚本，用这个脚本来构建、定制镜像，那么之前提及的无法重复的问题、镜像构建透明性的问题、体积的问题就都会解决。这个脚本就是 Dockerfile。

Dockerfile 是一个文本文件，其内包含了一条条的指令(Instruction)，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。

在一个空白目录中，建立一个文本文件，并命名为 Dockerfile：

	$ mkdir mynginx
	$ cd mynginx
	$ touch Dockerfile
	
其内容为：
	
	FROM nginx
	RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html

### FROM指定基础镜像

FROM 就是指定基础镜像，因此一个 Dockerfile 中 FROM 是必备的指令，并且必须是第一条指令。

Docker 还存在一个特殊的镜像，名为scratch 。这个镜像是虚拟的概念，并不实际存在，它表示一个空白的镜像。

	FROM scratch
	...
	
如果你以 scratch 为基础镜像的话，意味着你不以任何镜像为基础，接下来所写的指令将作为镜像第一层开始存在。

### RUN执行命令

两种格式：

*	shell格式：RUN <命令>
*	exec格式： RUN ["可执行文件", "参数1", "参数2"]，这更像是函数调用中的格式。

在撰写 Dockerfile 的时候，要经常提醒自己，这并不是在写 Shell 脚本，而是在定义每一层该如何构建。

Dockerfile 支持 Shell 类的行尾添加 \ 的命令换行方式，以及行首 # 进行注释的格式。

。因此镜像构建时，一定要确保每一层只添加真正需要添加的东西，任何无关的东西都应该清理掉。很多人初学 Docker 制作出了很臃肿的镜像的原因之一，就是忘记了每一层构建的最后一定要清理掉无关文件。

	FROM debian:jessie
	RUN buildDeps='gcc libc6-dev make' \
		&& apt-get update \
		&& apt-get install -y $buildDeps \
		&& wget -O redis.tar.gz "http://download.redis.io/releases/redis-3.2.5.tar.gz" \
		&& mkdir -p /usr/src/redis \
		&& tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1 \
		&& make -C /usr/src/redis \
		&& make -C /usr/src/redis install \
		&& rm -rf /var/lib/apt/lists/* \
		&& rm redis.tar.gz \
		&& rm -r /usr/src/redis \
		&& apt-get purge -y --auto-remove $buildDeps

### 构建镜像

在 Dockerfile 文件所在目录执行：
	
	$ docker build -t nginx:v3 .
	Sending build context to Docker daemon 2.048 kB
	Step 1 : FROM nginx
	---> e43d811ce2f4
	Step 2 : RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
	---> Running in 9cdc27646c7b
	---> 44aa4490ce2c
	Removing intermediate container 9cdc27646c7b
	Successfully built 44aa4490ce2c
	
### 镜像构建上下文(Context)

。当构建的时候，用户会指定构建镜像上下文的路径， docker build 命令得知这个路径后，会将路径下的所有内容打包，然后上传给 Docker 引擎。这样 Docker 引擎收到这个上下文包后，展开就会获得构建镜像所需的一切文件。

理解刚才的命令 docker build -t nginx:v3 . 中的这个 . ，实际上是在指定上下文的目录， docker build 命令会将该目录下的内容打包交给 Docker 引擎以帮助构建镜像。

	$ docker build -t nginx:v3 .
	Sending build context to Docker daemon 2.048 kB
	
一般来说，应该会将 Dockerfile 至于一个空目录下，或者项目根目录下。如果该目录下没有所需文件，那么应该把所需文件复制一份过来。如果目录下有些东西确实不希望构建时传给 Docker 引擎，那么可以用 .gitignore 一样的语法写一个 .dockerignore ，该文件是用于剔除不需要作为上下文传递给 Docker 引擎的。

### 其它docker build的用法

#### 直接用Git repo进行构建

	$ docker build https://github.com/twang2218/gitlab-ce-zh.git#:8.14
	
这行命令指定了构建所需的 Git repo，并且指定默认的 master 分支，构建目录为 /8.14/ ，然后 Docker 就会自己去 git clone 这个项目、切换到指定分支、并进入到指定目录后开始构建。

#### 用给定的tar压缩包构建

	$ docker build http://server/context.tar.gz

#### 从标准输入中读取Dockerfile进行构建

	docker build - < Dockerfile
	
或

	cat Dockerfile | docker build -

如果标准输入传入的是文本文件，则将其视为 Dockerfile ，并开始构建。这种形式由于直接从标准输入中读取 Dockerfile 的内容，它没有上下文，因此不可以像其他方法那样可以将本地文件 COPY 进镜像之类的事情。

#### 从标准输入中读取上下文压缩包进行构建

	$ docker build - < context.tar.gz

如果发现标准输入的文件格式是 gzip 、 bzip2 以及 xz 的话，将会使其为上下文压缩包，直接将其展开，将里面视为上下文，并开始构建。

## Dockerfile指令详解

### COPY复制文件

格式：

	*	COPY <源路径>... <目标路径>
	*	COPY ["<源路径1>",... "<目标路径>"]

	COPY package.json /usr/src/app/
	COPY hom* /mydir/
	COPY hom?.txt /mydir/
	
### ADD更高级的复制文件

如果 <源路径> 为一个 tar 压缩文件的话，压缩格式为 gzip , bzip2 以及 xz 的情况下， ADD 指令将会自动解压缩这个压缩文件到 <目标路径> 去。

	FROM scratch
	ADD ubuntu-xenial-core-cloudimg-amd64-root.tar.gz /
	
但在某些情况下，如果我们真的是希望复制个压缩文件进去，而不解压缩，这时就不可以使用 ADD 命令了。

在 COPY 和 ADD 指令中选择的时候，可以遵循这样的原则，所有的文件复制均使用 COPY 指令，仅在需要自动解压缩的场合使用 ADD 。

### CMD容器启动命令

格式：

*	shell 格式： CMD <命令>
*	exec 格式： CMD ["可执行文件", "参数1", "参数2"...]

CMD 指令就是用于指定默认的容器主进程的启动命令的。

在运行时可以指定新的命令来替代镜像设置中的这个默认命令，比如， ubuntu 镜像默认的 CMD 是 /bin/bash ，如果我们直接 docker run -it ubuntu 的话，会直接进入 bash 。我们也可以在运行时指定运行别的命令，如 docker run -it ubuntu cat /etc/os-release 。这就是用 cat /etc/os-release命令替换了默认的 /bin/bash 命令了，输出了系统版本信息。

在指令格式上，一般推荐使用 exec 格式，这类格式在解析时会被解析为 JSON 数组，因此一定要使用双引号 " ，而不要使用单引号。

如果使用 shell 格式的话，实际的命令会被包装为 sh -c 的参数的形式进行执行。比如：

	CMD echo $HOME
	
在实际执行中，会将其变更为：

	CMD [ "sh", "-c", "echo $HOME" ]

Docker 不是虚拟机，容器中的应用都应该以前台执行，而不是像虚拟机、物理机里面那样，用 upstart/systemd 去启动后台服务，容器内没有后台服务的概念。

一些初学者将 CMD 写为：

	CMD service nginx start
	
然后发现容器执行后就立即退出了。甚至在容器内去使用 systemctl 命令结果却发现根本执行不了。

对于容器而言，其启动程序就是容器应用进程，容器就是为了主进程而存在的，主进程退出，容器就失去了存在的意义，从而退出，其它辅助进程不是它需要关心的东西。

正确的做法是直接执行 nginx 可执行文件，并且要求以前台形式运行。比如：

	CMD ["nginx" "-g" "daemon off;"]

### ENTRYPOINT入口点

ENTRYPOINT 的格式和 RUN 指令格式一样，分为 exec 格式和 shell 格式。

ENTRYPOINT 的目的和 CMD 一样，都是在指定容器启动程序及参数。 ENTRYPOINT 在运行时也可以替代，不过比 CMD 要略显繁琐，需要通过 docker run 的参数 --entrypoint 来指定。

当指定了 ENTRYPOINT 后， CMD 的含义就发生了改变，不再是直接的运行其命令，而是将 CMD 的内容作为参数传给 ENTRYPOINT 指令。

#### 场景一：让镜像变成像命令一样使用

	FROM ubuntu:16.04
	RUN apt-get update \
		&& apt-get install -y curl \
		&& rm -rf /var/lib/apt/lists/*
	CMD [ "curl", "-s", "http://ip.cn" ]
	
假如我们使用 docker build -t myip . 来构建镜像的话，如果我们需要查询当前公网 IP，只需要执行：

	$ docker run myip

如果我们希望显示 HTTP 头信息，就需要加上 -i 参数。那么我们可以直接加 -i 参数给 docker run myip 么？

	$ docker run myip -i
	docker: Error response from daemon: invalid header field value "oci runtime error: container_linux.go:247: starting container process caused \"exec: \\\"-i\\\": executable file not found in $PATH\"\n".

这里的 -i 替换了远了的 CMD ，而不是添加在原来的 curl -s http://ip.cn 后面。而 -i 根本不是命令，所以自然找不到。

那么如果我们希望加入 -i 这参数，我们就必须重新完整的输入这个命令：

	$ docker run myip curl -s http://ip.cn -i
	
这显然不是很好的解决方案，而使用 ENTRYPOINT 就可以解决这个问题。

	FROM ubuntu:16.04
	RUN apt-get update \
		&& apt-get install -y curl \
		&& rm -rf /var/lib/apt/lists/*
	ENTRYPOINT [ "curl", "-s", "http://ip.cn" ]

这次我们再来尝试直接使用 docker run myip -i ：

	$ docker run myip -i
	
当存在 ENTRYPOINT 后， CMD 的内容将会作为参数传给 ENTRYPOINT ，而这里 -i 就是新的 CMD ，因此会作为参数传给 curl ，从而达到了我们预期的效果。

#### 场景二：应用运行前的准备工作

参考官方镜像redis的做法：

	FROM alpine:3.4
	...
	RUN addgroup -S redis && adduser -S -G redis redis
	...
	ENTRYPOINT ["docker-entrypoint.sh"]
	
	EXPOSE 6379
	CMD [ "redis-server" ]
	
可以看到其中为了 redis 服务创建了 redis 用户，并在最后指定了 ENTRYPOINT 为 docker-entrypoint.sh 脚本。

	#!/bin/sh
	...
	# allow the container to be started with `--user`
	if [ "$1" = 'redis-server' -a "$(id -u)" = '0' ]; then
		chown -R redis .
		exec su-exec redis "$0" "$@"
	fi
	
	exec "$@"
	
该脚本的内容就是根据 CMD 的内容来判断，如果是 redis-server 的话，则切换到 redis 用户身份启动服务器，否则依旧使用 root 身份执行。