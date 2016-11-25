---
layout: post
notes: true
subtitle: "高性能MySQL（第3版）"
comments: false
author: "【美】Baron Schwartz, Peter Zaitsev, Vadim Tkachenko 著（宁海元 周振兴 彭立勋 翟卫祥 等译）"
date: 2016-11-25 00:00:00

---

![](/img/notes/db/highPerformanceMySQL/high_performance_mysql.jpg)

*   目录
{:toc }

# 第1章 MySQL架构与历史

MySQL最重要、最与众不同的特性是它的存储引擎架构，这种架构的设计将查询处理（Query Processing）及其他系统任务（Server Task）和数据的存储/提取相分离。

## 1.1 MySQL逻辑架构

![](/img/notes/db/highPerformanceMySQL/logic_architect.png)

### 1.1.1 连接管理与安全性

每个客户端连接都会在服务器进程中拥有一个线程，服务器会负责缓存线程，因此不需要为每一个新建的连接创建或者销毁线程。

### 1.1.2 优化与执行

优化器并不关心表使用的是什么存储引擎，但存储引擎对于优化查询是有影响的。优化器会请求存储引擎提供容量或某个具体操作的开销信息，以及表数据的统计信息等。

## 1.2 并发控制

MySQL在两个层面的并发控制：服务器层与存储引擎层。