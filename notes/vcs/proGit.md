---
layout: post
notes: true
subtitle: Pro Git
comments: false
author: "Scott Chacon and Ben Straub"
date: 2017-01-24 00:00:00

---

![](/img/notes/vcs/proGit/progit.png)

[https://git-scm.com/book/en/v2](https://git-scm.com/book/en/v2)

*   目录
{:toc }

# 1. 起步

## 1.1 关于版本控制

版本控制是一种记录一个或若干文件内容变化，以便将来查阅特定版本修订情况的系统。

### 本地版本控制系统

![](/img/notes/vcs/proGit/local.png)

RCS：工作原理是在硬盘上保存补丁集（补丁是指文件修订前后的变化）；通过应用所有的补丁，可以重新计算出各个版本的文件内容。

### 集中化的版本控制系统

背景问题：如何让在不同系统上的开发者协同工作？

CCVS(Centralized Version Control Systems)：集中化的版本控制系统，诸如 CVS、Subversion 以及 Perforce 等，都有一个单一的集中管理的服务器，保存所有文件的修订版本，而协同工作的人们都通过客户端连到这台服务器，取出最新的文件或者提交更新。 多年以来，这已成为版本控制系统的标准做法。

![](/img/notes/vcs/proGit/centralized.png)

缺点：单点故障

### 分布式版本控制系统

DVCS(Distributed Version Control System)：分布式版本控制系统，这类系统中，像 Git、Mercurial、Bazaar 以及 Darcs 等，客户端并不只提取最新版本的文件快照，而是把代码仓库完整地镜像下来。 这么一来，任何一处协同工作用的服务器发生故障，事后都可以用任何一个镜像出来的本地仓库恢复。 因为每一次的克隆操作，实际上都是一次对代码仓库的完整备份。

![](/img/notes/vcs/proGit/distributed.png)

更进一步，许多这类系统都可以指定和若干不同的远端代码仓库进行交互。籍此，你就可以在同一个项目中，分别和不同工作小组的人相互协作。 你可以根据需要设定不同的协作流程，比如层次模型式的工作流，而这在以前的集中式系统中是无法实现的。

## 1.2 Git简史

目标：

*	速度
*	简单的设计
*	对非线性开发模式的强力支持（允许成千上万个并行开发的分支）
*	完全分布式
*	有能力高效管理类似 Linux 内核一样的超大规模项目（速度和数据量）

## 1.3 Git基础

### 直接记录快照，而非差异比较

![](/img/notes/vcs/proGit/deltas.png)

Git 不按照以上方式对待或保存数据。 反之，Git 更像是把数据看作是对小型文件系统的一组快照。 每次你提交更新，或在 Git 中保存项目状态时，它主要对当时的全部文件制作一个快照并保存这个快照的索引。 为了高效，如果文件没有修改，Git 不再重新存储该文件，而是只保留一个链接指向之前存储的文件。 Git 对待数据更像是一个 **快照流**。

![](/img/notes/vcs/proGit/snapshots.png)

### 近乎所有操作都是本地执行

在 Git 中的绝大多数操作都只需要访问本地文件和资源，一般不需要来自网络上其它计算机的信息。举个例子，要浏览项目的历史，Git 不需外连到服务器去获取历史，然后再显示出来——它只需直接从本地数据库中读取。

### Git保证完整性

Git 中所有数据在存储前都计算校验和，然后以校验和来引用。 这意味着不可能在 Git 不知情时更改任何文件内容或目录内容。 这个功能建构在 Git 底层，是构成 Git 哲学不可或缺的部分。 若你在传送过程中丢失信息或损坏文件，Git 就能发现。

Git 用以计算校验和的机制叫做 SHA-1 散列（hash，哈希）。

### Git一般只添加数据

你执行的 Git 操作，几乎只往 Git 数据库中增加数据。 很难让 Git 执行任何不可逆操作，或者让它以任何方式清除数据。 同别的 VCS 一样，未提交更新时有可能丢失或弄乱修改的内容；但是一旦你提交快照到 Git 中，就难以再丢失数据，特别是如果你定期的推送数据库到其它仓库的话。

这使得我们使用 Git 成为一个安心愉悦的过程，因为我们深知可以尽情做各种尝试，而没有把事情弄糟的危险。

### 三种状态

*	已提交（committed）：表示数据已经安全的保存在本地数据库中。
*	已修改（modified）：表示修改了文件，但还没保存到数据库中。
*	已暂存（staged）：表示对一个已修改文件的当前版本做了标记，使之包含在下次提交的快照中。

由此引入 Git 项目的三个工作区域的概念：Git 仓库、工作目录以及暂存区域。

![](/img/notes/vcs/proGit/areas.png)

*	Git 仓库目录是 Git 用来保存项目的元数据和对象数据库的地方。 这是 Git 中最重要的部分，从其它计算机克隆仓库时，拷贝的就是这里的数据。
*	工作目录是对项目的某个版本独立提取出来的内容。 这些从 Git 仓库的压缩数据库中提取出来的文件，放在磁盘上供你使用或修改。
*	暂存区域是一个文件，保存了下次将提交的文件列表信息，一般在 Git 仓库目录中。 有时候也被称作`‘索引’'，不过一般说法还是叫暂存区域。

基本的 Git 工作流程如下：

1.	在工作目录中修改文件。
2.	暂存文件，将文件的快照放入暂存区域。
3.	提交更新，找到暂存区域的文件，将快照永久性存储到 Git 仓库目录。

如果 Git 目录中保存着的特定版本文件，就属于已提交状态。 如果作了修改并已放入暂存区域，就属于已暂存状态。 如果自上次取出后，作了修改但还没有放到暂存区域，就是已修改状态。

## 1.4 命令行

只有在命令行模式下你才能执行 Git 的 所有 命令，而大多数的 GUI 软件只实现了 Git 所有功能的一个子集以降低操作难度

## 1.5 安装Git

### 在Linux上安装

Fedora：

	$ sudo yum install git
	
Debian：

	$ sudo apt-get install git
	
要了解更多选择，Git 官方网站上有在各种 Unix 风格的系统上安装步骤，网址为 http://git-scm.com/download/linux。

### 在Mac上安装

在 Mac 上安装 Git 有多种方式。 最简单的方法是安装 Xcode Command Line Tools。 Mavericks （10.9） 或更高版本的系统中，在 Terminal 里尝试首次运行 git 命令即可。 如果没有安装过命令行开发者工具，将会提示你安装。

如果你想安装更新的版本，可以使用二进制安装程序。 官方维护的 OSX Git 安装程序可以在 Git 官方网站下载，网址为 http://git-scm.com/download/mac。

### 在Windows上安装

msysGit：http://git-scm.com/download/win、http://msysgit.github.io/

Git for Windows：http://windows.github.com

### 从源代码安装

可以使用以下命令之一来安装最小化的依赖包来编译和安装 Git 的二进制版：

	$ sudo yum install curl-devel expat-devel gettext-devel \
		openssl-devel zlib-devel
	$ sudo apt-get install libcurl4-gnutls-dev libexpat1-dev gettext \
		libz-dev libssl-dev

为了能够添加更多格式的文档（如 doc, html, info），你需要安装以下的依赖包：

	$ sudo yum install asciidoc xmlto docbook2x
	$ sudo apt-get install asciidoc xmlto docbook2x
	
当你安装好所有的必要依赖，你可以继续从几个地方来取得最新发布版本的 tar 包。 你可以从 Kernel.org 网站获取，网址为 https://www.kernel.org/pub/software/scm/git，或从 GitHub 网站上的镜像来获得，网址为 https://github.com/git/git/releases。 通常在 GitHub 上的是最新版本，但 kernel.org 上包含有文件下载签名，如果你想验证下载正确性的话会用到。

编译并安装：

	$ tar -zxf git-2.0.0.tar.gz
	$ cd git-2.0.0
	$ make configure
	$ ./configure --prefix=/usr
	$ make all doc info
	$ sudo make install install-doc install-html install-info
	
完成后，你可以使用 Git 来获取 Git 的升级：

	$ git clone git://git.kernel.org/pub/scm/git/git.git

## 1.6 起步 - 初次运行Git前的配置

Git 自带一个 git config 的工具来帮助设置控制 Git 外观和行为的配置变量。 这些变量存储在三个不同的位置：

1.	/etc/gitconfig 文件: 包含系统上每一个用户及他们仓库的通用配置。 如果使用带有 --system 选项的 git config 时，它会从此文件读写配置变量。
2.	~/.gitconfig 或 ~/.config/git/config 文件：只针对当前用户。 可以传递 --global 选项让 Git 读写此文件。
3.	当前使用仓库的 Git 目录中的 config 文件（就是 .git/config）：针对该仓库。

每一个级别覆盖上一级别的配置，所以 .git/config 的配置变量会覆盖 /etc/gitconfig 中的配置变量

### 用户信息

当安装完 Git 应该做的第一件事就是设置你的用户名称与邮件地址。 这样做很重要，因为每一个 Git 的提交都会使用这些信息，并且它会写入到你的每一次提交中，不可更改：

	$ git config --global user.name "John Doe"
	$ git config --global user.email johndoe@example.com
	
再次强调，如果使用了 --global 选项，那么该命令只需要运行一次，因为之后无论你在该系统上做任何事情，Git 都会使用那些信息。 当你想针对特定项目使用不同的用户名称与邮件地址时，可以在那个项目目录下运行没有 --global 选项的命令来配置。

### 文本编辑器

既然用户信息已经设置完毕，你可以配置默认文本编辑器了，当 Git 需要你输入信息时会调用它。 如果未配置，Git 会使用操作系统默认的文本编辑器，通常是 Vim。 如果你想使用不同的文本编辑器，例如 Emacs，可以这样做：

	$ git config --global core.editor emacs
	
### 检查配置信息

如果想要检查你的配置，可以使用 git config --list 命令来列出所有 Git 当时能找到的配置。

	$ git config --list
	user.name=John Doe
	user.email=johndoe@example.com
	color.status=auto
	color.branch=auto
	color.interactive=auto
	color.diff=auto
	...
	
你可能会看到重复的变量名，因为 Git 会从不同的文件中读取同一个配置（例如：/etc/gitconfig 与 ~/.gitconfig）。 这种情况下，Git 会使用它找到的每一个变量的最后一个配置。

你可以通过输入 git config <key>： 来检查 Git 的某一项配置
	
	$ git config user.name
	John Doe

## 1.7 获取帮助

若你使用 Git 时需要获取帮助，有三种方法可以找到 Git 命令的使用手册：

	$ git help <verb>
	$ git <verb> help
	$ man git-<verb>
	
例如，要想获得 config 命令的手册，执行

	$ git help config
	
## 1.8 总结

# 2. Git基础

## 2.1 获取Git仓库

有两种取得 Git 项目仓库的方法。 第一种是在现有项目或目录下导入所有文件到 Git 中； 第二种是从一个服务器克隆一个现有的 Git 仓库。

### 在现有目录中初始化仓库

如果你打算使用 Git 来对现有的项目进行管理，你只需要进入该项目目录并输入：

	$ git init
	
该命令将创建一个名为 .git 的子目录，这个子目录含有你初始化的 Git 仓库中所有的必须文件，这些文件是 Git 仓库的骨干。 但是，在这个时候，我们仅仅是做了一个初始化的操作，你的项目里的文件还没有被跟踪。

如果你是在一个已经存在文件的文件夹（而不是空文件夹）中初始化 Git 仓库来进行版本控制的话，你应该开始跟踪这些文件并提交。 你可通过 git add 命令来实现对指定文件的跟踪，然后执行 git commit 提交：

	$ git add *.c
	$ git add LICENSE
	$ git commit -m 'initial project version'
	
现在，你已经得到了一个实际维护（或者说是跟踪）着若干个文件的 Git 仓库。

### 克隆现有的仓库

	$ git clone https://github.com/libgit2/libgit2
	
这会在当前目录下创建一个名为 “libgit2” 的目录，并在这个目录下初始化一个 .git 文件夹，从远程仓库拉取下所有数据放入 .git 文件夹，然后从中读取最新版本的文件的拷贝。 如果你进入到这个新建的 libgit2 文件夹，你会发现所有的项目文件已经在里面了，准备就绪等待后续的开发和使用。 如果你想在克隆远程仓库的时候，自定义本地仓库的名字，你可以使用如下命令：

	$ git clone https://github.com/libgit2/libgit2 mylibgit
	
这将执行与上一个命令相同的操作，不过在本地创建的仓库名字变为 mylibgit。

Git 支持多种数据传输协议。上面的例子使用的是 https:// 协议，不过你也可以使用 git:// 协议或者使用 SSH 传输协议，比如 user@server:path/to/repo.git 。