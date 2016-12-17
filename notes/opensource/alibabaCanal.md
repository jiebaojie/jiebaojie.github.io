---
layout: post
notes: true
subtitle: "【开源代码】Canal：阿里巴巴mysql数据库binlog的增量订阅&消费组件"
comments: false
author: "Alibaba Open Source"
date: 2016-12-17 00:00:00

---

GitHub地址：[https://github.com/alibaba/canal/](https://github.com/alibaba/canal/)

*   目录
{:toc }

# 背景

早期，阿里巴巴B2B公司因为存在杭州和美国双机房部署，存在跨机房同步的业务需求。不过早期的数据库同步业务，主要是基于trigger的方式获取增量变更，不过从2010年开始，阿里系公司开始逐步的尝试基于数据库的日志解析，获取增量变更进行同步，由此衍生出了增量订阅&消费的业务，从此开启了一段新纪元。

当前的canal开源版本支持5.7及以下的版本(阿里内部mysql 5.7.13, 5.6.10, mysql 5.5.18和5.1.40/48)

基于日志增量订阅&消费支持的业务：

1.	数据库镜像
2.	数据库实时备份
3.	多级索引（卖家和买家各自分库索引）
4.	search build
5.	业务cache刷新
6.	价格变化等重要业务消息

# 项目介绍

## 工作原理

### mysql主备复制实现

![](/img/notes/opensource/alibabaCanal/mysql_replication.jpg)

从上层来看，复制分成三步：

1.	master将改变记录到二进制日志(binary log)中（这些记录叫做二进制日志事件，binary log events，可以通过show binlog events进行查看）；
2.	slave将master的binary log events拷贝到它的中继日志(relay log)；
3.	slave重做中继日志中的事件，将改变反映它自己的数据。

### canal工作原理：

![](/img/notes/opensource/alibabaCanal/canal.jpg)

原理相对比较简单：

1.	canal模拟mysql slave的交互协议，伪装自己为mysql slave，向mysql master发送dump协议
2.	mysql master收到dump请求，开始推送binary log给slave(也就是canal)
3.	canal解析binary log对象(原始为byte流)

# 环境要求

## 1. 操作系统

*	纯java开发，windows/linux均可支持
*	jdk建议使用1.6.25以上的版本，稳定可靠，目前阿里巴巴使用基本为此版本。

## 2. mysql要求

目前canal测试已支持mysql 5.7.13/5.6.10及以下的版本，mariadb 5.5.35和10.0.7(理论上可支持以下版本)

canal的原理是基于mysql binlog技术，所以这里一定需要开启mysql的binlog写入功能，建议配置binlog模式为row.(ps. 目前canal已经支持mixed/statement/row模式)
 
	[mysqld]
	log-bin=mysql-bin #添加这一行就ok  
	binlog-format=ROW #选择row模式  
	server_id=1 #配置mysql replaction需要定义，不能和canal的slaveId重复
	
canal的原理是模拟自己为mysql slave，所以这里一定需要做为mysql slave的相关权限：

	CREATE USER canal IDENTIFIED BY 'canal';    
	GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'canal'@'%';  
	-- GRANT ALL PRIVILEGES ON *.* TO 'canal'@'%' ;  
	FLUSH PRIVILEGES;
	
针对已有的账户可通过grants查询权限：

	show grants for 'canal';
	
# 架构

![](/img/notes/opensource/alibabaCanal/architect.jpg)

说明：

*	server代表一个canal运行实例，对应于一个jvm
*	instance对应于一个数据队列 （1个server对应1..n个instance)

instance模块：

*	eventParser (数据源接入，模拟slave协议和master进行交互，协议解析)
*	eventSink (Parser和Store链接器，进行数据过滤，加工，分发的工作)
*	eventStore (数据存储)
*	metaManager (增量订阅&消费信息管理器)

## 知识科普：

mysql的Binlay Log介绍：

*	[http://dev.mysql.com/doc/refman/5.5/en/binary-log.html](http://dev.mysql.com/doc/refman/5.5/en/binary-log.html)
*	[http://www.taobaodba.com/html/474_mysqls-binary-log_details.html](http://www.taobaodba.com/html/474_mysqls-binary-log_details.html)

简单点说：

*	mysql的binlog是多文件存储，定位一个LogEvent需要通过binlog filename + binlog position，进行定位
*	mysql的binlog数据格式，按照生成的方式，主要分为：statement-based、row-based、mixed。

	mysql> show variables like 'binlog_format';
	
目前canal支持所有模式的增量订阅(但配合同步时，因为statement只有sql，没有数据，无法获取原始的变更日志，所以一般建议为ROW模式)

## EventParser设计

整个parser过程分为：

1.	Connection获取上一次解析成功的位置 (如果第一次启动，则获取初始指定的位置或者是当前数据库的binlog位点)
2.	Connection建立链接，发送BINGLOG_DUMP指令
	*	0. write command number
	*	1. write 4 bytes bin-log position to start at
	*	2. write 2 bytes bin-log flags
	*	3. write 4 bytes server id of the slave
	*	4. write bin-log file name
3.	Mysql开始推送Binaly Log
4.	接收到的Binaly Log的通过Binlog parser进行协议解析，补充一些特定信息（补充字段名字，字段类型，主键信息，unsigned类型处理）
5.	传递给EventSink模块进行数据存储，是一个阻塞操作，直到存储成功
6.	存储成功后，定时记录Binaly Log位置

mysql的Binary Log网络协议：

![](/img/notes/opensource/alibabaCanal/mysql_binary_log.png)

说明：

*	图中的协议4byte header，主要是描述整个binlog网络包的length
*	binlog event structure，详细信息请参考： [http://forge.mysql.com/wiki/MySQL_Internals_Binary_Log](http://forge.mysql.com/wiki/MySQL_Internals_Binary_Log)


