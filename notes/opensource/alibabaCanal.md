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

## EventSink设计

![](/img/notes/opensource/alibabaCanal/eventsink.jpg)

说明：

*	数据过滤：支持通配符的过滤模式，表名，字段内容等
*	数据路由/分发：解决1:n (1个parser对应多个store的模式)
*	数据归并：解决n:1 (多个parser对应1个store)
*	数据加工：在进入store之前进行额外的处理，比如join

## EventStore设计

1.	目前仅实现了Memory内存模式，后续计划增加本地file存储，mixed混合模式
2.	借鉴了Disruptor的RingBuffer的实现思路

RingBuffer设计：

![](/img/notes/opensource/alibabaCanal/ringbuffer.jpg)

定义了3个cursor：

*	Put : Sink模块进行数据存储的最后一次写入位置
*	Get : 数据订阅获取的最后一次提取位置
*	Ack : 数据消费成功的最后一次消费位置

借鉴Disruptor的RingBuffer的实现，将RingBuffer拉直来看：

![](/img/notes/opensource/alibabaCanal/ringbuffer2.jpg)

实现说明：

*	Put/Get/Ack cursor用于递增，采用long型存储
*	buffer的get操作，通过取余或者与操作。(与操作： cusor & (size - 1) , size需要为2的指数，效率比较高)

## Instance设计

instance代表了一个实际运行的数据队列，包括了EventPaser,EventSink,EventStore等组件。

抽象了CanalInstanceGenerator，主要是考虑配置的管理方式：

*	manager方式：和你自己的内部web console/manager系统进行对接。(目前主要是公司内部使用)
*	spring方式：基于spring xml + properties进行定义，构建spring配置。

## Server设计

![](/img/notes/opensource/alibabaCanal/server.jpg)

server代表了一个canal的运行实例，为了方便组件化使用，特意抽象了Embeded(嵌入式) / Netty(网络访问)的两种实现

*	Embeded : 对latency和可用性都有比较高的要求，自己又能hold住分布式的相关技术(比如failover)
*	Netty : 基于netty封装了一层网络协议，由canal server保证其可用性，采用的pull模型，当然latency会稍微打点折扣，不过这个也视情况而定。

## 增量订阅/消费设计

![](/img/notes/opensource/alibabaCanal/subscribe_consumer.jpg)

get/ack/rollback协议介绍：

*	Message getWithoutAck(int batchSize)，允许指定batchSize，一次可以获取多条，每次返回的对象为Message，包含的内容为：batch id唯一标识、entries 具体的数据对象。
*	void rollback(long batchId)，顾命思议，回滚上次的get请求，重新获取数据。基于get获取的batchId进行提交，避免误操作。
*	void ack(long batchId)，顾命思议，确认已经消费成功，通知server删除数据。基于get获取的batchId进行提交，避免误操作。

canal的get/ack/rollback协议和常规的jms协议有所不同，允许get/ack异步处理，比如可以连续调用get多次，后续异步按顺序提交ack/rollback，项目中称之为流式api。

流式api设计的好处：

*	get/ack异步化，减少因ack带来的网络延迟和操作成本 (99%的状态都是处于正常状态，异常的rollback属于个别情况，没必要为个别的case牺牲整个性能)
*	get获取数据后，业务消费存在瓶颈或者需要多进程/多线程消费时，可以不停的轮询get数据，不停的往后发送任务，提高并行化。

流式api设计：

![](/img/notes/opensource/alibabaCanal/flow_api.jpg)

*	每次get操作都会在meta中产生一个mark，mark标记会递增，保证运行过程中mark的唯一性
*	每次的get操作，都会在上一次的mark操作记录的cursor继续往后取，如果mark不存在，则在last ack cursor继续往后取
*	进行ack时，需要按照mark的顺序进行数序ack，不能跳跃ack. ack会删除当前的mark标记，并将对应的mark位置更新为last ack cusor
*	一旦出现异常情况，客户端可发起rollback情况，重新置位：删除所有的mark, 清理get请求位置，下次请求会从last ack cursor继续往后取

## HA机制设计

canal的ha分为两部分，canal server和canal client分别有对应的ha实现：

*	canal server: 为了减少对mysql dump的请求，不同server上的instance要求同一时间只能有一个处于running，其他的处于standby状态。
*	canal client: 为了保证有序性，一份instance同一时间只能由一个canal client进行get/ack/rollback操作，否则客户端接收无法保证有序。

整个HA机制的控制主要是依赖了zookeeper的几个特性，watcher和EPHEMERAL节点(和session生命周期绑定)。

Canal Server：

![](/img/notes/opensource/alibabaCanal/ha_server.jpg)

大致步骤：

1.	canal server要启动某个canal instance时都先向zookeeper进行一次尝试启动判断(实现：创建EPHEMERAL节点，谁创建成功就允许谁启动)
2.	创建zookeeper节点成功后，对应的canal server就启动对应的canal instance，没有创建成功的canal instance就会处于standby状态
3.	一旦zookeeper发现canal server A创建的节点消失后，立即通知其他的canal server再次进行步骤1的操作，重新选出一个canal server启动instance。
4.	canal client每次进行connect时，会首先向zookeeper询问当前是谁启动了canal instance，然后和其建立链接，一旦链接不可用，会重新尝试connect。

Canal Client的方式和canal server方式类似，也是利用zookeeper的抢占EPHEMERAL节点的方式进行控制。
