---
layout: post
notes: true
subtitle: "深入理解MariaDB与MySQL"
comments: false
author: "【韩】李成旭 著（武传海 译）"
date: 2016-11-24 00:00:00

---

![](/img/notes/db/deepUnderstandingMariaDBMySQL/deep_understanding_mariadb_mysql.jpg)

*   目录
{:toc }

# 第1章 MariaDB

## 1.1 MariaDB

1.  MariaDB诞生于MySQL Community代码数据库。
2.  MariaDB是由Monty Program AB维护的MySQL。
3.  MariaDB是开源数据库。

## 1.2 MariaDB与MySQL

### 1.2.1 MariaDB、MySQL和PerconaServer

PerconaServer是Percona公司（http://www.percona.com/）以MySQL服务器源代码为基础创建的MySQL服务器的另一个分支版本。

PerconaServer不会使用MariaDB中实现的新功能，但是MariaDB会把Percona中实现的功能一起添加到新的发布版本，其中最具代表性的就是Percona开发的XtraDB存储引擎。

XtraDB是在InnoDB的源码基础上改良而成的新存储引擎。XtraDB与InnoDB的所有数据文件保持100%兼容。因此，实际应用中完全可以使用XtraDB取代InnoDB存储引擎。

![](/img/notes/db/deepUnderstandingMariaDBMySQL/mariadb_mysql_perconaserver_version.jpg)

### 1.2.2 相同点

*   MariaDB的执行程序、实用工具与MySQL同名且互相兼容。
*   MySQL 5.x的数据文件与.FRM文件（表定义文件）与MariaDB 5.x兼容。
*   所有客户端API与通信协议相互兼容。
*   所有文件（与复制相关的数据文件、套接字文件）、端口及文件路径一致。
*   MySQL Connector（Java驱动程序及C客户端库文件）可以直接在MariaDB中使用。
*   MySQL客户端程序可以直接用于连接MariaDB服务器。

MariaDB 10.0与MySQL 5.6中，用于实现复制的二进制日志格式略有差别。

在服务器优化器，存储引擎的设置内容方面，MariaDB 5.x与MySQL有着明显不同。而MariaDB 10.x版本中，这些区别更加明显。

从MariaDB 10.x开始，面向开发人员的部分在易用性、效率、性能方面有了很大改善，与MySQL相比逐渐有了一些差异。