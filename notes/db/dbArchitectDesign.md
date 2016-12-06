---
layout: post
notes: true
subtitle: "【技术博客】数据库软件架构设计些什么"
comments: false
author: "58沈剑 架构师之路"
date: 2016-12-04 00:00:00

---


原文：[http://mp.weixin.qq.com/s?__biz=MjM5ODYxMDA5OQ==&mid=400465735&idx=1&sn=8d7067de4cc8f73ea5558f07e0a9340e](http://mp.weixin.qq.com/s?__biz=MjM5ODYxMDA5OQ==&mid=400465735&idx=1&sn=8d7067de4cc8f73ea5558f07e0a9340e)

*   目录
{:toc }

# 一、基本概念

## 概念一：单库

![](/img/notes/db/dbArchitectDesign/single_base.png)

## 概念二：分片

![](/img/notes/db/dbArchitectDesign/split.png)

分片解决的是“数据量太大”的问题，也就是通常说的“水平切分”。

一旦引入分片，势必有“数据路由”的概念，哪个数据访问哪个库。

路由规则的3种方法：

*   范围：range
    *   优点：简单，容易扩展
    *   缺点：各库压力不均（新号段更活跃）
*   哈希：hash
    *   优点：简单，数据均衡，负载均匀
    *   缺点：迁移麻烦（2库扩3库数据要迁移）
*   路由服务：router-config-server
    *   优点：灵活性强，业务与路由算法解耦
    *   缺点：每次访问数据库前多一次查询

大部分互联网公司采用的方案二：哈希分库，哈希路由

## 概念三：分组

![](/img/notes/db/dbArchitectDesign/group.png)

分组解决“可用性”问题，分组通常通过主从复制的方式实现。

互联网公司数据库实际软件架构是：又分片，又分组

![](/img/notes/db/dbArchitectDesign/split_group.png)

# 二、数据库架构设计思路

考虑以下四点：

*   如何保证数据可用性
*   如何提高数据库读性能（大部分应用读多写少，读会先成为瓶颈）
*   如何保证一致性
*   如何提高扩展性

## 2.1 如何保证数据的可用性？

**解决可用性问题的思路是=>冗余**

*   如何保证站点的可用性？复制站点，冗余站点
*   如何保证服务的可用性？复制服务，冗余服务
*   如何保证数据的可用性？复制数据，冗余数据

**数据的冗余，会带来一个副作用=>引发一致性问题**

### 如何保证数据库"读"高可用？

冗余读库

![](/img/notes/db/dbArchitectDesign/read_high_available.png)

缺点：

*   读写有延迟，可能不一致
*   写仍然是单点，不能保证写高可用

### 如何保证数据库"写"高可用？

冗余写库

![](/img/notes/db/dbArchitectDesign/write_high_available.png)

采用双主互备的方式，可以冗余写库

副作用？双写同步，数据可能冲突（例如“自增id”同步冲突）,如何解决同步冲突，有两种常见解决方案：

*   两个写库使用不同的初始值，相同的步长来增加id：1写库的id为0,2,4,6...；2写库的id为1,3,5,7...
*   不使用数据的id，业务层自己生成唯一的id，保证数据不冲突

### 58方案：**双主当主从用**

![](/img/notes/db/dbArchitectDesign/double_master.png)

仍是双主，但只有一个主提供服务（读+写），另一个主是“shadow-master”，只用来保证高可用，平时不提供服务。master挂了，shadow-master顶上（vip漂移，对业务层透明，不需要人工介入）

好处：

*   读写没有延时
*   读写高可用

不足：

*   不能通过加从库的方式扩展读性能
*   资源利用率为50%，一台冗余主没有提供服务

## 2.2 如何扩展读性能？

### 建立索引

不同的库可以建立不同的索引。

![](/img/notes/db/dbArchitectDesign/index.png)

*   写库不建立索引；
*   线上读库建立线上访问索引，例如uid；
*   线下读库建立线下访问索引，例如time；

### 增加从库

缺点：

*   从库越多，同步越慢
*   同步越慢，数据不一致窗口越大

### 58方案：**增加缓存**

常见缓存架构：

![](/img/notes/db/dbArchitectDesign/cache_data.png)

上游是业务应用，下游是从哭，从库（读写分离），缓存。

58玩法：**服务+数据库+缓存一套**

![](/img/notes/db/dbArchitectDesign/58_db_cache.png)

业务层不直接面向db和cache，服务层屏蔽了底层db、cache的复杂性

不管采用主从的方式扩展读性能，还是缓存的方式扩展读性能，数据都要复制多份

（主+从，db+cache），一定会引发一致性问题。

## 2.3 如何保证一致性？

### 主从数据库一致性

#### 中间件

![](/img/notes/db/dbArchitectDesign/middleware.png)

如果某一个key有写操作，在不一致时间窗口内，中间件会将这个key的读操作也路由到主库上。

缺点：数据库中间件的门槛较高

#### 强制读主

![](/img/notes/db/dbArchitectDesign/double_master.png)

58的“双主当主从用”的架构，不存在主从不一致的问题。

### db与缓存间的不一致

![](/img/notes/db/dbArchitectDesign/cache_data.png)

常见的缓存架构如上，此时写操作的顺序是：

1.  淘汰cache
2.  写数据库

读操作的顺序是：

1.  读cache，如果cache hit则返回
2.  如果cache miss，则读从库
3.  读从库后，将数据放回cache

在一些异常时序情况下，有可能从【从库读到旧数据（同步还没有完成），旧数据入cache后】，数据会长期不一致。

**解决办法是“缓存双淘汰”**，写操作时序升级为：

1.  淘汰cache
2.  写数据库
3.  在经验“主从同步延时窗口时间”后，再次发起一个异步淘汰cache的请求

这样，即使有脏数据如cache，一个小的时间窗口之后，脏数据还是会被淘汰。带来的代价是，多引入一次读miss（成本可以忽略）。

**建议为所有cache中的item设置一个超时时间。**

## 2.4 如何提高数据库的扩展性？

原来用hash的方式路由，分为2个库，数据量还是太大，要分为3个库，势必需要进行数据迁移。

如何秒级扩容？

首先，我们不做2库变3库的扩容，我们做2库变4库（库加倍）的扩容（未来4->8->16）

![](/img/notes/db/dbArchitectDesign/two_db.png)

服务+数据库是一套（省去了缓存），数据库采用“双主”的模式。

**扩容步骤**：

*   第一步，将一个主库提升
*   第二步，修改配置，2库变4库（原来MOD2，现在配置修改后MOD4），扩容完成

![](/img/notes/db/dbArchitectDesign/scale_up.png)

原MOD2为偶的部分，现在会MOD4余0或者2，原MOD2为奇的部分，现在会MOD4余1或者3

数据不需要迁移，同时，双主互相同步，一遍是余0，一边余2，两边数据同步也不会冲突，秒级完成扩容！

最后，要做一些收尾工作：

1.  将旧的双主同步解除
2.  增加新的双主（双主是保证可用性的，shadow-master平时不提供服务）
3.  删除多余的数据（余0的主，可以将余2的数据删除掉）

![](/img/notes/db/dbArchitectDesign/tail.png)

