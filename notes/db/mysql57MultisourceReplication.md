---
layout: post
notes: true
subtitle: "【技术博客】性能提升利器：MySQL 5.7多源主从复制的独特性"
comments: false
author: "高强(EnweiTech)"
date: 2017-01-10 00:00:00

---


原文：[http://blog.csdn.net/enweitech/article/details/53082228](http://blog.csdn.net/enweitech/article/details/53082228)

*   目录
{:toc }

# 关于MySQL主从复制

## 一些主从复制架构：

![](/img/notes/db/mysql57MultisourceReplication/replication_architect.png)

![](/img/notes/db/mysql57MultisourceReplication/replication_architect.jpeg)

## 主从复制的优势：

1.	分担负载：对业务进行读写分离，减轻主库I/O负载，将部分压力分担到从库上，缩短客户查询响应时间。
2.	增加健壮性：在主库出现问题时，可通过多种方案将从库设置为主库，替换主库支撑业务，缩短停机窗口。
3.	有利备份：在从库上备份，即不影响主库的事务，也不影响主库性能和磁盘空间。
4.	查询分析：从库可以作为统计、报表等数据分析工作所使用的的OLAP库。
5.	异地备份：将从库放置在异地可作为异地数据同步备份所用。

从MySQL的5.7版本开始支持多源主从复制技术（Multi-Source Replication），就是将多个数据库（Master）的数据集中发送到1台从库（Slave）上，其优势在于：

1.	汇聚数据：尤其是在分库分表的一些场景中，数据集中统计分析操作可以在1台从库服务器上实现。
2.	节省成本：数据集中存放可避免服务器等软硬件资源浪费，5.7之前1主1从或者1主多从的方案需要为每个主机都安置一台备机；5.7推出多源复制之后，可以将多个从库进行合并，至于是合并存放在高端还是低端服务器上，取决于分析、统计等业务在整体业务中的优先级、繁忙程度等因素。
3.	集中备份：方便在一台服务器备份所有已收到的数据库数据。
4.	异地灾备：将从库放在距离远的地方，可用于异地备份项目。

## 复制过程

![](/img/notes/db/mysql57MultisourceReplication/one_one_replication.png)

图中Master为主库的主机名，Slave为从库主机名。同步的数据库名为Music。从库接收主库（binlog dump线程）发过来的Binary Log，通过从库的I/O线程（I/O thread）将日志写入从库的Relay Log中，然后通过SQL线程（SQL thread）将日志的内容应用到从库中。

从库上的2个必备进程(I/O thread和SQL thread)在待命状态：

*	I/O线程：Waiting for master to send event
*	SQL(Coordinator)线程：Slave has read all relay log; waiting for more updates

开启并发后还会有以下线程：

*	Worker线程：Waiting for an event from Coordinator

多源复制的实现与1主1从的类似，都是发送二进制日志再重演，但是在SQL线程（SQL thread）上有略微区别，会为每个主库实例提供一套SQL和IO线程。

多源复制的配置：

1.	stop slave;
2.	SET GLOBAL master_info_repository = 'TABLE';
3.	SET GLOBAL relay_log_info_repository = 'TABLE';
4.	change master to master_host='192.168.5.160',master_user='slave1',master_password='gaoqiang' for channel 'master1';
5.	change master to master_host='192.168.5.163',master_user='slave1',master_password='gaoqiang' for channel 'master2';
6.	start slave;

![](/img/notes/db/mysql57MultisourceReplication/multisource_replication.jpeg)

考虑到多源复制运行过程中，一台从库要接受多方的数据，相比压力会比单库复制有所提升，因此需要优化一下从库配置，从而提升从库执行效率。

## 性能提升利器——并行复制

将事务在从库上多线程并发的回放应用，从而达到加速同步速度的效果。

1主1从SQL线程并行复制回放过程：

![](/img/notes/db/mysql57MultisourceReplication/concurrent_replication.png)

SQL线程（SQL thread）由原来的1个，分裂变成了1个Coordinator和N个worker。Coordinator主要用来分发工作给不同的worker，并且在必要时自己也进行重演操作。图中分了18个并行，即18个worker并行工作。他们负责并行的将日志中可以划分成一组的事务进行并行回放。

在把事务应用到从库的时候，根据binary log中last_committed（需设置并行类型为logical clock）的值判断是否可以放在一组进行并行回放，如果取值相同，便并行执行。

多源复制开启并发后的架构图：

![](/img/notes/db/mysql57MultisourceReplication/multisource_concurrent_replication.jpeg)

开启并行复制操作的方法对于1主和多源是一样的：

1.	stop slave;
2.	SET GLOBAL SLAVE_PARALLEL_TYPE='LOGICAL_CLOCK';
3.	SET GLOBAL SLAVE_PARALLEL_WORKERS=30;
4.	start slave;

验证配置生效：

	mysql> show variables like '%SLAVE_PARALLEL%';
	
Sysbench压力测试

多源复制在合理的开启并行之后，有助于提高复制效率，缩短数据的延迟。

## 小结

总体说来，MySQL的多源复制提供了更经济、方便和安全的数据库环境。