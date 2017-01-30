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

## 2.2 Git基础 - 记录每次更新到仓库

工作目录下的每一个文件都不外乎这两种状态：已跟踪或未跟踪。 已跟踪的文件是指那些被纳入了版本控制的文件，在上一次快照中有它们的记录，在工作一段时间后，它们的状态可能处于未修改，已修改或已放入暂存区。 工作目录中除已跟踪文件以外的所有其它文件都属于未跟踪文件，它们既不存在于上次快照的记录中，也没有放入暂存区。 初次克隆某个仓库的时候，工作目录中的所有文件都属于已跟踪文件，并处于未修改状态。

![](/img/notes/vcs/proGit/lifecycle.png)

### 检查当前文件状态

要查看哪些文件处于什么状态，可以用 git status 命令。 如果在克隆仓库后立即使用此命令，会看到类似这样的输出：

	$ git status
	On branch master
	nothing to commit, working directory clean

这说明你现在的工作目录相当干净。换句话说，所有已跟踪文件在上次提交后都未被更改过。 此外，上面的信息还表明，当前目录下没有出现任何处于未跟踪状态的新文件，否则 Git 会在这里列出来。 最后，该命令还显示了当前所在分支，并告诉你这个分支同远程服务器上对应的分支没有偏离。 现在，分支名是 “master”，这是默认的分支名。

现在，让我们在项目下创建一个新的 README 文件。 如果之前并不存在这个文件，使用 git status 命令，你将看到一个新的未跟踪文件：

	$ echo 'My Project' > README
	$ git status
	On branch master
	Untracked files:
	 (use "git add <file>..." to include in what will be committed)
	 
		README
		
	nothing added to commit but untracked files present (use "git add" to track)
	
### 跟踪新文件

	$ git add README
	
此时再运行 git status 命令，会看到 README 文件已被跟踪，并处于暂存状态：

	$ git status
	On branch master
	Changes to be committed:
	 (use "git reset HEAD <file>..." to unstage)
	 
		new file:   README

只要在 Changes to be committed 这行下面的，就说明是已暂存状态。 如果此时提交，那么该文件此时此刻的版本将被留存在历史记录中。

git add 命令使用文件或目录的路径作为参数；如果参数是目录的路径，该命令将递归地跟踪该目录下的所有文件。

### 暂存已修改的文件

	$ git status
	On branch master
	Changes to be committed:
	 (use "git reset HEAD <file>..." to unstage)
	 
		new file:   README
		
	Changes not staged for commit:
	 (use "git add <file>..." to update what will be committed)
	 (use "git checkout -- <file>..." to discard changes in working directory)
	 
		modified:   CONTRIBUTING.md
		
文件 CONTRIBUTING.md 出现在 Changes not staged for commit 这行下面，说明已跟踪文件的内容发生了变化，但还没有放到暂存区。 要暂存这次更新，需要运行 git add 命令。 这是个多功能命令：可以用它开始跟踪新文件，或者把已跟踪的文件放到暂存区，还能用于合并时把有冲突的文件标记为已解决状态等。 将这个命令理解为“添加内容到下一次提交中”而不是“将一个文件添加到项目中”要更加合适。 现在让我们运行 git add 将"CONTRIBUTING.md"放到暂存区，然后再看看 git status 的输出：

	$ git add CONTRIBUTING.md
	$ git status
	On branch master
	Changes to be committed:
	 (use "git reset HEAD <file>..." to unstage)
	 
		new file:   README
		modified:   CONTRIBUTING.md
		
现在两个文件都已暂存，下次提交时就会一并记录到仓库。 假设此时，你想要在 CONTRIBUTING.md 里再加条注释， 重新编辑存盘后，准备好提交。 不过且慢，再运行 git status 看看：

	$ git add CONTRIBUTING.md
	$ git status
	On branch master
	Changes to be committed:
	 (use "git reset HEAD <file>..." to unstage)
	 
		new file:   README
		modified:   CONTRIBUTING.md
		
	Changes not staged for commit:
	 (use "git add <file>..." to update what will be committed)
	 (use "git checkout -- <file>..." to discard changes in working directory)
	 
		modified:   CONTRIBUTING.md
		
现在 CONTRIBUTING.md 文件同时出现在暂存区和非暂存区。实际上 Git 只不过暂存了你运行 git add 命令时的版本， 如果你现在提交，CONTRIBUTING.md 的版本是你最后一次运行 git add 命令时的那个版本，而不是你运行 git commit 时，在工作目录中的当前版本。 所以，运行了 git add 之后又作了修订的文件，需要重新运行 git add 把最新版本重新暂存起来：

	$ git add CONTRIBUTING.md
	$ git status
	On branch master
	Changes to be committed:
	 (use "git reset HEAD <file>..." to unstage)
	 
		new file:   README
		modified:   CONTRIBUTING.md
		
### 状态简览

git status 命令的输出十分详细，但其用语有些繁琐。 如果你使用 git status -s 命令或 git status --short 命令，你将得到一种更为紧凑的格式输出。 运行 git status -s ，状态报告输出如下：

	$ git status -s
	M README
	MM Rakefile
	A  lib/git.rb
	M  lib/simplegit.rb
	?? LICENSE.txt
	
新添加的未跟踪文件前面有 ?? 标记，新添加到暂存区中的文件前面有 A 标记，修改过的文件前面有 M 标记。 你可能注意到了 M 有两个可以出现的位置，出现在右边的 M 表示该文件被修改了但是还没放入暂存区，出现在靠左边的 M 表示该文件被修改了并放入了暂存区。 例如，上面的状态报告显示： README 文件在工作区被修改了但是还没有将修改后的文件放入暂存区,lib/simplegit.rb 文件被修改了并将修改后的文件放入了暂存区。 而 Rakefile 在工作区被修改并提交到暂存区后又在工作区中被修改了，所以在暂存区和工作区都有该文件被修改了的记录。

### 忽略文件

一般我们总会有些文件无需纳入 Git 的管理，也不希望它们总出现在未跟踪文件列表。 通常都是些自动生成的文件，比如日志文件，或者编译过程中创建的临时文件等。 在这种情况下，我们可以创建一个名为 .gitignore 的文件，列出要忽略的文件模式。

	$ cat .gitignore
	*.[oa]
	*~
	
第一行告诉 Git 忽略所有以 .o 或 .a 结尾的文件。一般这类对象文件和存档文件都是编译过程中出现的。 第二行告诉 Git 忽略所有以波浪符（~）结尾的文件，许多文本编辑软件（比如 Emacs）都用这样的文件名保存副本。

文件.gitignore的格式规范如下：

	*	所有空行或者以 ＃ 开头的行都会被 Git 忽略。
	*	可以使用标准的 glob 模式匹配。
	*	匹配模式可以以（/）开头防止递归
	*	匹配模式可以以（/）结尾指定目录。
	*	要忽略指定模式以外的文件或目录，可以在模式前加上惊叹号（!）取反。
	
所谓的 glob 模式是指 shell 所使用的简化了的正则表达式。 星号（*）匹配零个或多个任意字符；[abc] 匹配任何一个列在方括号中的字符（这个例子要么匹配一个 a，要么匹配一个 b，要么匹配一个 c）；问号（?）只匹配一个任意字符；如果在方括号中使用短划线分隔两个字符，表示所有在这两个字符范围内的都可以匹配（比如 [0-9] 表示匹配所有 0 到 9 的数字）。 使用两个星号（*) 表示匹配任意中间目录，比如`a/**/z` 可以匹配 a/z, a/b/z 或 `a/b/c/z`等。

	# no .a files
	*.a
	
	# but do track lib.a, even though you're ignoring .a files above
	!lib.a
	
	# only ignore the TODO in the current directory, not subdir/TODO
	/TODO
	
	# ignore all files in the build/ directory
	build/
	
	# ignore doc/notes.txt, but not doc/server/arch.txt
	doc/*.txt

	# ignore all .pdf files in the doc/ directory
	doc/**/*.pdf
	
GitHub 有一个十分详细的针对数十种项目及语言的 .gitignore 文件列表，你可以在 https://github.com/github/gitignore 找到它。

### 查看已暂存和未暂存的修改

git diff 将通过文件补丁的格式显示具体哪些行发生了改变。

要查看尚未暂存的文件更新了哪些部分，不加参数直接输入 git diff：

	$ git diff
	
若要查看已暂存的将要添加到下次提交里的内容，可以用 git diff --cached 命令。（Git 1.6.1 及更高版本还允许使用 git diff --staged，效果是相同的，但更好记些。）

	$ git diff --staged
	
请注意，git diff 本身只显示尚未暂存的改动，而不是自上次提交以来所做的所有改动。 所以有时候你一下子暂存了所有更新过的文件后，运行 git diff 后却什么也没有，就是这个原因。

如果你喜欢通过图形化的方式或其它格式输出方式的话，可以使用 git difftool 命令来用 Araxis ，emerge 或 vimdiff 等软件输出 diff 分析结果。 使用 git difftool --tool-help 命令来看你的系统支持哪些 Git Diff 插件。

### 提交更新

现在的暂存区域已经准备妥当可以提交了。 在此之前，请一定要确认还有什么修改过的或新建的文件还没有 git add 过，否则提交的时候不会记录这些还没暂存起来的变化。 这些修改过的文件只保留在本地磁盘。 所以，每次准备提交前，先用 git status 看下，是不是都已暂存起来了， 然后再运行提交命令 git commit：

	$ git commit
	
这种方式会启动文本编辑器以便输入本次提交的说明

另外，你也可以在 commit 命令后添加 -m 选项，将提交信息与命令放在同一行，如下所示：

	$ git commit -m "Story 182: Fix benchmarks for speed"
	
可以看到，提交后它会告诉你，当前是在哪个分支（master）提交的，本次提交的完整 SHA-1 校验和是什么（463dc4f），以及在本次提交中，有多少文件修订过，多少行添加和删改过。

请记住，提交时记录的是放在暂存区域的快照。 任何还未暂存的仍然保持已修改状态，可以在下次提交时纳入版本管理。 每一次运行提交操作，都是对你项目作一次快照，以后可以回到这个状态，或者进行比较。

### 跳过使用暂存区域

尽管使用暂存区域的方式可以精心准备要提交的细节，但有时候这么做略显繁琐。 Git 提供了一个跳过使用暂存区域的方式， 只要在提交的时候，给 git commit 加上 -a 选项，Git 就会自动把所有已经跟踪过的文件暂存起来一并提交，从而跳过 git add 步骤：

	$ git commit -a -m 'added new benchmarks'
	
### 移除文件

要从 Git 中移除某个文件，就必须要从已跟踪文件清单中移除（确切地说，是从暂存区域移除），然后提交。 可以用 git rm 命令完成此项工作，并连带从工作目录中删除指定的文件，这样以后就不会出现在未跟踪文件清单中了。

如果只是简单地从工作目录中手工删除文件，运行 git status 时就会在 “Changes not staged for commit” 部分（也就是 未暂存清单）看到：

	$ rm PROJECTS.md
	$ git status
	On branch master
	Your branch is up-to-date with 'origin/master'.
	Changes not staged for commit:
	 (use "git add/rm <file>..." to update what will be committed)
	 (use "git checkout -- <file>..." to discard changes in working directory)
	 
		deleted:    PROJECTS.md
		
	no changes added to commit (use "git add" and/or "git commit -a")
	
然后再运行 git rm 记录此次移除文件的操作：

	$ git rm PROJECTS.md
	rm 'PROJECTS.md'
	$ git status
	On branch master
	Changes to be committed:
	 (use "git reset HEAD <file>..." to unstage)
	 
		deleted:    PROJECTS.md
		
下一次提交时，该文件就不再纳入版本管理了。 如果删除之前修改过并且已经放到暂存区域的话，则必须要用强制删除选项 -f（译注：即 force 的首字母）。 这是一种安全特性，用于防止误删还没有添加到快照的数据，这样的数据不能被 Git 恢复。

另外一种情况是，我们想把文件从 Git 仓库中删除（亦即从暂存区域移除），但仍然希望保留在当前工作目录中。 换句话说，你想让文件保留在磁盘，但是并不想让 Git 继续跟踪。 当你忘记添加 .gitignore 文件，不小心把一个很大的日志文件或一堆 .a 这样的编译生成文件添加到暂存区时，这一做法尤其有用。 为达到这一目的，使用 --cached 选项：

	$ git rm --cached README
	
git rm 命令后面可以列出文件或者目录的名字，也可以使用 glob 模式。 比方说：

	$ git rm log/\*.log
	
注意到星号 * 之前的反斜杠 \， 因为 Git 有它自己的文件模式扩展匹配方式，所以我们不用 shell 来帮忙展开。 此命令删除 log/ 目录下扩展名为 .log 的所有文件。 类似的比如：

	$ git rm \*~
	
该命令为删除以 ~ 结尾的所有文件。

### 移动文件

要在 Git 中对文件改名，可以这么做：
 
	$ git mv file_from file_to
	
它会恰如预期般正常工作。 实际上，即便此时查看状态信息，也会明白无误地看到关于重命名操作的说明：

	$ git mv README.md README
	$ git status
	On branch master
	Changes to be committed:
	 (use "git reset HEAD <file>..." to unstage)
	 
		renamed:    README.md -> README
		
其实，运行git mv 就相当于运行了下面三条命令：

	$ mv README.md README
	$ git rm README.md
	$ git add README
	
如此分开操作，Git 也会意识到这是一次改名，所以不管何种方式结果都一样。 两者唯一的区别是，mv 是一条命令而另一种方式需要三条命令，直接用 git mv 轻便得多。 不过有时候用其他工具批处理改名的话，要记得在提交前删除老的文件名，再添加新的文件名。

## 2.3 查看提交历史

在提交了若干更新，又或者克隆了某个项目之后，你也许想回顾下提交历史。完成这个任务最简单而又有效的工具是 git log 命令。

	$ git log
	
默认不用任何参数的话，git log 会按提交时间列出所有的更新，最近的更新排在最上面。 正如你所看到的，这个命令会列出每个提交的 SHA-1 校验和、作者的名字和电子邮件地址、提交时间以及提交说明。

一个常用的选项是 -p，用来显示每次提交的内容差异。 你也可以加上 -2 来仅显示最近两次提交：

	$ git log -p -2
	
该选项除了显示基本信息之外，还附带了每次 commit 的变化。 当进行代码审查，或者快速浏览某个搭档提交的 commit 所带来的变化的时候，这个参数就非常有用了。 你也可以为 git log 附带一系列的总结性选项。 比如说，如果你想看到每次提交的简略的统计信息，你可以使用 --stat 选项：

	$ git log --stat
	
正如你所看到的，--stat 选项在每次提交的下面列出额所有被修改过的文件、有多少文件被修改了以及被修改过的文件的哪些行被移除或是添加了。 在每次提交的最后还有一个总结。

另外一个常用的选项是 --pretty。 这个选项可以指定使用不同于默认格式的方式展示提交历史。 这个选项有一些内建的子选项供你使用。 比如用 oneline 将每个提交放在一行显示，查看的提交数很大时非常有用。 另外还有 short，full 和 fuller 可以用，展示的信息或多或少有些不同：

	$ git log --pretty=oneline
	
但最有意思的是 format，可以定制要显示的记录格式。 这样的输出对后期提取分析格外有用 — 因为你知道输出的格式不会随着 Git 的更新而发生改变：

	$ git log --pretty=format:"%h - %an, %ar : %s"
	ca82a6d - Scott Chacon, 6 years ago : changed the version number
	085bb3b - Scott Chacon, 6 years ago : removed unnecessary test
	a11bef0 - Scott Chacon, 6 years ago : first commit

git log --pretty=format常用的选项：

*	%H：提交对象（commit）的完整哈希字串
*	%h：提交对象的简短哈希字串
*	%T：树对象（tree）的完整哈希字串
*	%t：树对象的简短哈希字串
*	%P：父对象（parent）的完整哈希字串
*	%p：父对象的简短哈希字串
*	%an：作者（author）的名字
*	%ae：作者的电子邮件地址
*	%ad：作者修订日期（可以用 --date= 选项定制格式）
*	%ar：作者修订日期，按多久以前的方式显示
*	%cn：提交者（committer）的名字
*	%ce：提交者的电子邮件地址
*	%cd：提交日期
*	%cr：提交日期，按多久以前的方式显示
*	%s：提交说明

你一定奇怪 作者 和 提交者 之间究竟有何差别， 其实作者指的是实际作出修改的人，提交者指的是最后将此工作成果提交到仓库的人。 所以，当你为某个项目发布补丁，然后某个核心成员将你的补丁并入项目时，你就是作者，而那个核心成员就是提交者。

当 oneline 或 format 与另一个 log 选项 --graph 结合使用时尤其有用。 这个选项添加了一些ASCII字符串来形象地展示你的分支、合并历史：

	$ git log --pretty=format:"%h %s" --graph
	
git log 的常用选项：

*	-p：按补丁格式显示每个更新之间的差异。
*	--stat：显示每次更新的文件修改统计信息。
*	--shortstat：只显示 --stat 中最后的行数修改添加移除统计。
*	--name-only：仅在提交信息后显示已修改的文件清单。
*	--name-status：显示新增、修改、删除的文件清单。
*	--abbrev-commit：仅显示 SHA-1 的前几个字符，而非所有的 40 个字符。
*	--relative-date：使用较短的相对时间显示（比如，“2 weeks ago”）。
*	--graph：显示 ASCII 图形表示的分支合并历史。
*	--pretty：使用其他格式显示历史提交信息。可用的选项包括 oneline，short，full，fuller 和 format（后跟指定格式）。

### 限制输出长度

Git 在输出所有提交时会自动调用分页程序，所以你一次只会看到一页的内容。

另外还有按照时间作限制的选项，比如 --since 和 --until 也很有用。 例如，下面的命令列出所有最近两周内的提交：

	$ git log --since=2.weeks
	
还可以给出若干搜索条件，列出符合的提交。 用 --author 选项显示指定作者的提交，用 --grep 选项搜索提交说明中的关键字。 （请注意，如果要得到同时满足这两个选项搜索条件的提交，就必须用 --all-match 选项。否则，满足任意一个条件的提交都会被匹配出来）

另一个非常有用的筛选选项是 -S，可以列出那些添加或移除了某些字符串的提交。 比如说，你想找出添加或移除了某一个特定函数的引用的提交，你可以这样使用：

	$ git log -Sfunction_name
	
最后一个很实用的 git log 选项是路径（path）， 如果只关心某些文件或者目录的历史提交，可以在 git log 选项的最后指定它们的路径。 因为是放在最后位置上的选项，所以用两个短划线（--）隔开之前的选项和后面限定的路径名。

限制 git log 输出的选项：

*	-(n)：仅显示最近的 n 条提交
*	--since, --after：仅显示指定时间之后的提交。
*	--until, --before：仅显示指定时间之前的提交。
*	--author：仅显示指定作者相关的提交。
*	--committer：仅显示指定提交者相关的提交。
*	--grep：仅显示含指定关键字的提交
*	-S：仅显示添加或移除了某个关键字的提交

来看一个实际的例子，如果要查看 Git 仓库中，2008 年 10 月期间，Junio Hamano 提交的但未合并的测试文件，可以用下面的查询命令：

	$ git log --pretty="%h - %s" --author=gitster --since="2008-10-01" --before="2008-11-01" --no-merges -- t/