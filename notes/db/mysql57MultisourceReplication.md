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

一些主从复制架构：

![](/img/notes/db/mysql57MultisourceReplication/replication_architect.png)

![](/img/notes/db/mysql57MultisourceReplication/replication_architect.jpeg)

主从复制的优势：

1.	分担负载：对业务进行读写分离，减轻主库I/O负载，将部分压力分担到从库上，缩短客户查询响应时间。
2.	增加健壮性：在主库出现问题时，可通过多种方案将从库设置为主库，替换主库支撑业务，缩短停机窗口。
3.	有利备份：在从库上备份，即不影响主库的事务，也不影响主库性能和磁盘空间。
4.	查询分析：从库可以作为统计、报表等数据分析工作所使用的的OLAP库。
5.	异地备份：将从库放置在异地可作为异地数据同步备份所用。

