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