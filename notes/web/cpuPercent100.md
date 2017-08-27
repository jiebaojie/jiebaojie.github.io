---
layout: post
notes: true
subtitle: "【技术博客】线上服务CPU100%问题快速定位实战"
comments: false
author: "架构师之路"
date: 2017-08-27 00:00:00

---


原文：[https://mp.weixin.qq.com/s/Xb1im4jG_Cobhas4q4YT1Q](https://mp.weixin.qq.com/s/Xb1im4jG_Cobhas4q4YT1Q)

*   目录
{:toc }

# 功能问题

通过日志，单步调试相对比较好定位。

# 性能问题

例如线上服务器CPU100%，如何找到相关服务，如何定位问题代码，更考验技术人的功底。

# 题目

某服务器上部署了若干tomcat实例，即若干垂直切分的Java站点服务，以及若干Java微服务，突然收到运维的CPU异常告警。

问：如何定位是哪个服务进程导致CPU过载，哪个线程导致CPU过载，哪段代码导致CPU过载？

# 步骤一、找到最耗CPU的进程

工具：top

方法：

*	执行top -c ，显示进程运行信息列表
*	键入P (大写p)，进程按照CPU使用率排序

如：找到最耗CPU的进程PID为10765

# 步骤二、找到最耗CPU的线程

工具：top

方法：

*	top -Hp 10765，显示一个进程的线程运行信息列表
*	键入P（大写p），线程按照CPU使用率排序

如：进程10765内，最耗CPU的线程PID为10804

# 步骤三、将线程PID转化为16进制

工具：printf

方法：printf “%x\n” 10804

如：10804对应的16进制是0x2a34，当然，这一步可以用计算器。

之所以要转化为16进制，是因为堆栈里，线程id是用16进制表示的。

# 步骤四：查看堆栈，找到线程在干嘛

工具：pstack/jstack/grep

方法：

	jstack 10765 | grep ‘0x2a34’ -C5 --color

*	打印进程堆栈
*	通过线程id，过滤得到线程堆栈


