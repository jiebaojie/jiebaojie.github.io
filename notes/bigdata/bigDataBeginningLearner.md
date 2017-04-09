---
layout: post
notes: true
subtitle: "【技术博客】写给大数据开发初学者的话"
comments: false
author: "lxw1234"
date: 2017-04-09 00:00:00

---


原文1：[http://lxw1234.com/archives/2016/11/779.htm](http://lxw1234.com/archives/2016/11/779.htm)

原文2：[http://lxw1234.com/archives/2016/11/782.htm](http://lxw1234.com/archives/2016/11/782.htm)

原文3：[http://lxw1234.com/archives/2016/11/787.htm](http://lxw1234.com/archives/2016/11/787.htm)

*   目录
{:toc }

# 导读

大数据的三个发展方向，平台搭建/优化/运维/监控、大数据开发/设计/架构、数据分析/挖掘。

大数据的4V特征：

*	数据量大，TB->PB
*	数据类型繁多，结构化、非结构化文本、日志、视频、图片、地理位置等；
*	商业价值高，但是这种价值需要在海量数据之上，通过数据分析与机器学习更快速的挖掘出来；
*	处理时效性高，海量数据的处理需求不再局限在离线计算当中。

现如今，正式为了应对大数据的这几个特点，开源的大数据框架越来越多，越来越强，先列举一些常见的：

*	文件存储：Hadoop HDFS、Tachyon、KFS
*	离线计算：Hadoop MapReduce、Spark
*	流式、实时计算：Storm、Spark Streaming、S4、Heron
*	K-V、NOSQL数据库：HBase、Redis、MongoDB
*	资源管理：YARN、Mesos
*	日志收集：Flume、Scribe、Logstash、Kibana
*	消息系统：Kafka、StormMQ、ZeroMQ、RabbitMQ
*	查询分析：Hive、Impala、Pig、Presto、Phoenix、SparkSQL、Drill、Flink、Kylin、Druid
*	分布式协调服务：Zookeeper
*	集群管理与监控：Ambari、Ganglia、Nagios、Cloudera Manager
*	数据挖掘、机器学习：Mahout、Spark MLLib
*	数据同步：Sqoop
*	任务调度：Oozie

# 第一章：初识Hadoop

## 1.1 学会百度与Google

## 1.2 参考资料首选官方文档

## 1.3 先让Hadoop跑起来

Hadoop可以算是大数据存储和计算的开山鼻祖，现在大多开源的大数据框架都依赖Hadoop或者与它能很好的兼容。

*	Hadoop 1.0、Hadoop 2.0
*	MapReduce、HDFS
*	NameNode、DataNode
*	JobTracker、TaskTracker
*	Yarn、ResourceManager、NodeManager

## 1.4 试试使用Hadoop

*	HDFS目录操作命令；
*	提交运行MapReduce示例程序；
*	打开Hadoop WEB界面，查看Job运行状态，查看Job运行日志。
*	知道Hadoop的系统日志在哪里。

## 1.5 你该了解它们的原理了

*	MapReduce：如何分而治之；
*	HDFS：数据到底在哪里，什么是副本；
*	Yarn到底是什么，它能干什么；
*	NameNode到底在干些什么；
*	ResourceManager到底在干些什么；

## 1.6 自己写一个MapReduce程序

# 第二章：更高效的WordCount

## 2.1 学点SQL吧

## 2.2 SQL版WordCount

## 2.3 SQL On Hadoop之Hive

什么是Hive？

The Apache Hive data warehouse software facilitates reading, writing, and managing large datasets residing in distributed storage and queried using SQL syntax.

为什么说Hive是数据仓库工具，而不是数据库工具呢？有的朋友可能不知道数据仓库，数据仓库是逻辑上的概念，底层使用的是数据库，数据仓库中的数据有这两个特点：最全的历史数据（海量）、相对稳定的；所谓相对稳定，指的是数据仓库不同于业务系统数据库，数据经常会被更新，数据一旦进入数据仓库，很少会被更新和删除，只会被大量查询。而Hive，也是具备这两个特点，因此，Hive适合做海量数据的数据仓库工具，而不是数据库工具。

## 2.4 安装配置Hive

## 2.5 试试使用Hive

## 2.6 Hive是怎么工作的

明明写的是SQL，为什么Hadoop WEB界面中看到的是MapReduce任务？

## 2.7 学会Hive的基本命令

*	创建、删除表；
*	加载数据到表；
*	下载Hive表的数据；
*	学习更多关于Hive的语法和命令。

目前已经具备的技能和知识点：

*	0和Hadoop2.0的区别；
*	MapReduce的原理
*	HDFS读写数据的流程；向HDFS中PUT数据；从HDFS中下载数据；
*	自己会写简单的MapReduce程序，运行出现问题，知道在哪里查看日志；
*	会写简单的SELECT、WHERE、GROUP BY等SQL语句；
*	Hive SQL转换成MapReduce的大致流程；
*	Hive中常见的语句：创建表、删除表、往表中加载数据、分区、将表中数据下载到本地；

HDFS是Hadoop提供的分布式存储框架，它可以用来存储海量数据，MapReduce是Hadoop提供的分布式计算框架，它可以用来统计和分析HDFS上的海量数据，而Hive则是SQL On Hadoop，Hive提供了SQL接口，开发人员只需要编写简单易上手的SQL语句，Hive负责把SQL翻译成MapReduce，提交运行。

此时，你的”大数据平台”是这样的：

![](/img/notes/bigdata/bigDataBeginningLearner/hadoop_hive.jpg)

那么问题来了，海量数据如何到HDFS上呢？

# 第三章：把别处的数据搞到Hadoop上

此处也可以叫做数据采集，把各个数据源的数据采集到Hadoop上。

## 3.1 HDFS PUT命令

## 3.2 HDFS API

HDFS提供了写数据的API，自己用编程语言将数据写入HDFS，put命令本身也是使用API。

实际环境中一般自己较少编写程序使用API来写数据到HDFS，通常都是使用其他框架封装好的方法。比如：Hive中的INSERT语句，Spark中的saveAsTextfile等。

## 3.3 Sqoop

Sqoop是一个主要用于Hadoop/Hive与传统关系型数据库Oracle/MySQL/SQLServer等之间进行数据交换的开源框架。

就像Hive把SQL翻译成MapReduce一样，Sqoop把你指定的参数翻译成MapReduce，提交到Hadoop运行，完成Hadoop与其他数据库之间的数据交换。

自己下载和配置Sqoop（建议先使用Sqoop1，Sqoop2比较复杂）。

了解Sqoop常用的配置参数和方法。

使用Sqoop完成从MySQL同步数据到HDFS；

使用Sqoop完成从MySQL同步数据到Hive表；

## 3.4 Flume

Flume是一个分布式的海量日志采集和传输框架，因为“采集和传输框架”，所以它并不适合关系型数据库的数据采集和传输。

Flume可以实时的从网络协议、消息系统、文件系统采集日志，并传输到HDFS上。

因此，如果你的业务有这些数据源的数据，并且需要实时的采集，那么就应该考虑使用Flume。

*	下载和配置Flume。
*	使用Flume监控一个不断追加数据的文件，并将数据传输到HDFS；

## 3.5 阿里开源的DataX

参考：[异构数据源海量数据交换工具-Taobao DataX 下载和使用](http://lxw1234.com/archives/2015/05/231.htm)

现在DataX已经是3.0版本，支持很多数据源。

如果你认真完成了上面的学习和实践，此时，你的”大数据平台”应该是这样的：

![](/img/notes/bigdata/bigDataBeginningLearner/collector.jpg)

# 第四章：把Hadoop上的数据搞到别处去

## 4.1 HDFS GET命令

## 4.2 HDFS API

## 4.3 Sqoop

*	使用Sqoop完成将HDFS上的文件同步到MySQL；
*	使用Sqoop完成将Hive表中的数据同步到MySQL；

## 4.4 DataX

如果你认真完成了上面的学习和实践，此时，你的”大数据平台”应该是这样的：

![](/img/notes/bigdata/bigDataBeginningLearner/export.jpg)

已经具备以下技能和知识点：

*	知道如何把已有的数据采集到HDFS上，包括离线采集和实时采集；
*	你已经知道sqoop（或者还有DataX）是HDFS和其他数据源之间的数据交换工具；
*	你已经知道flume可以用作实时的日志采集；

从前面的学习，对于大数据平台，你已经掌握的不少的知识和技能，搭建Hadoop集群，

把数据采集到Hadoop上，使用Hive和MapReduce来分析数据，把分析结果同步到其他数据源。

接下来的问题来了，Hive使用的越来越多，你会发现很多不爽的地方，特别是速度慢，大多情况下，明明我的数据量很小，它都要申请资源，启动MapReduce来执行。

# 第五章：快一点吧，我的SQL

其实大家都已经发现Hive后台使用MapReduce作为执行引擎，实在是有点慢。

因此SQL On Hadoop的框架越来越多，按我的了解，最常用的按照流行度依次为SparkSQL、Impala和Presto.

这三种框架基于半内存或者全内存，提供了SQL接口来快速查询分析Hadoop上的数据。

使用SparkSQL的原因：

*	使用Spark还做了其他事情，不想引入过多的框架；
*	Impala对内存的需求太大，没有过多资源部署；

## 5.1 关于Spark和SparkSQL

*	什么是Spark，什么是SparkSQL。
*	Spark有的核心概念及名词解释。
*	SparkSQL和Spark是什么关系，SparkSQL和Hive是什么关系。
*	SparkSQL为什么比Hive跑的快。

## 5.2 如何部署和运行SparkSQL

*	Spark有哪些部署模式？
*	如何在Yarn上运行SparkSQL？
*	使用SparkSQL查询Hive中的表。

关于Spark和SparkSQL，可参考 [http://lxw1234.com/archives/category/spark](http://lxw1234.com/archives/category/spark)

如果你认真完成了上面的学习和实践，此时，你的”大数据平台”应该是这样的：

![](/img/notes/bigdata/bigDataBeginningLearner/spark.jpg)