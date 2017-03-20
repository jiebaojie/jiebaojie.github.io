---
layout: post
notes: true
subtitle: "【技术博客】1分钟实现“延迟消息”功能"
comments: false
author: "58沈剑（架构师之路）"
date: 2017-03-19 00:00:00

---


原文：[http://mp.weixin.qq.com/s?__biz=MjM5ODYxMDA5OQ==&mid=2651959961&idx=1&sn=afec02c8dc6db9445ce40821b5336736](http://mp.weixin.qq.com/s?__biz=MjM5ODYxMDA5OQ==&mid=2651959961&idx=1&sn=afec02c8dc6db9445ce40821b5336736)

*   目录
{:toc }

# 一、缘起

很多时候，业务有“在一段时间之后，完成一个工作任务”的需求。

常见方案：启动一个cron定时任务，每小时跑一次。如果数据量很大，需要分页查询，分页update，这将会是一个for循环。

方案的不足：

1.	轮询效率比较低
2.	每次扫库，已经被执行过记录，仍然会被扫描（只是不会出现在结果集中），有重复计算的嫌疑
3.	时效性不够好，如果每小时轮询一次，最差的情况下，时间误差会达到1小时
4.	如果通过增加cron轮询频率来减少3中的时间误差，1中轮询低效和2中重复计算的问题会进一步凸显。

# 二、高效延时消息设计与实现

高效延时消息，包含两个重要的数据结构：

1.	环形队列，例如可以创建一个包含3600个slot的环形队列（本质是个数组）
2.	任务集合，环上每一个slot是一个Set<Task>

同时，启动一个timer，这个timer每隔1s，在上述环形队列中移动一格，有一个Current Index指针来标识正在检测的slot。

Task结构中有两个很重要的属性：

1.	Cycle-Num：当Current Index第几圈扫描到这个Slot时，执行任务
2.	Task-Function：需要执行的任务指针

假设当前Current Index指向第一格，当有延时消息到达之后，例如希望3610秒之后，触发一个延时消息任务，只需：

1.	计算这个Task应该放在哪一个slot，现在指向1，3610秒之后，应该是第11格，所以这个Task应该放在第11个slot的Set<Task>中
2.	计算这个Task的Cycle-Num，由于环形队列是3600格（每秒移动一格，正好1小时），这个任务是3610秒后执行，所以应该绕3610/3600=1圈之后再执行，于是Cycle-Num=1

Current Index不停的移动，每秒移动到一个新slot，这个slot中对应的Set<Task>，每个Task看Cycle-Num是不是0：

1.	如果不是0，说明还需要多移动几圈，将Cycle-Num减1
2.	如果是0，说明马上要执行这个Task了，取出Task-Funciton执行（可以用单独的线程来执行Task），并把这个Task从Set<Task>中删除

# 三、总结

环形队列是一个实现“延时消息”的好方法，开源的MQ好像都不支持延迟消息，不妨自己实现一个简易的“延时消息队列”，能解决很多业务问题，并减少很多低效扫库的cron任务。

# 四、其他参考

*	[laravel的消息队列剖析](http://www.cnblogs.com/yjf512/p/6571941.html)