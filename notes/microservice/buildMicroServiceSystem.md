---
layout: post
notes: true
subtitle: "【技术博客】利用开源社区打造微服务生态体系"
comments: false
author: "敖小剑"
date: 2016-12-03 00:00:00

---


原文：[http://sanwen.net/a/uwmxypo.html](http://sanwen.net/a/uwmxypo.html)

*   目录
{:toc }

# 前言

三部分内容：

*   微服务的核心技术
*   目前可选的开源微服务框架
*   为微服务提供支撑的基础设施

# 第一部分：核心技术

四个主要技术：

*   进程间通讯
*   服务注册与发现
*   负载均衡
*   熔断

其他技术：

*   故障转移（Failover）
*   失败重试
*   快速失败（Failfast）

重要概念：在微服务架构中，为了彻底隔绝不同服务，采用了最坚决的方案，强制要求服务之间：通过 **远程访问** 方式进行通讯。

在这点上，微服务和以OSGi、jigsaw为代表的Java模块化方案形成鲜明对比。

进程间通讯的方式比较多，其多样性体现在两个方面：

*   有三种风格的解决方案：REST，RPC 和 定制
*   交互方式有两个维度：按照交互对象的数量分为一对一和一对多，按照应答返回的方式分为同步和异步。

网络类库Netty版本的选择：

*   Netty3.*：老版本，推荐升级，新项目不要选
*   Netty4.0.*：主流版本，追求稳定的选择
*   Netty5.*：因为 ForkJoinPool 引入了太多复杂度而又未能带来明确的性能提升，**已废弃版本**，切记不要选！已上贼船的赶紧下来
*   Netty4.1：最新版本，支持HTTP/2，有需要再上

Rest框架比较大众化的选择：

*   服务器端Spring MVC
*   客户端用Jersey

RPC的选择：

*   怀旧的同学请选择hessian
*   求稳的同学请选择thrift
*   追新的同学请选择gRPC

注意：如果需要支持移动设备，如果想要用HTTTP 2 的新特性，那么就只能选择gRPC了。

消息队列的选择（个人建议）

*   轻量级使用 选择RabbitMQ
*   重量级使用 选择Apache Kafka
*   要求非常高 考虑一下NSQ
*   没有特殊原因，不要选择ActiveMQ

服务注册和服务发现，在实现时根据对一致性要求的不同，分成两个流派：

*   强一致性：比较常见的分布式一致性协议是 PAXOS 协议和 Raft 协议。相比 PAXOS 而言，Raft 协议易于理解和实现，因此最新的分布式一致性方案大都选择 Raft 协议。 zookeeper 采用的是 PAXOS 协议(实际为改进版本ZAP)，而 Raft 协议那边主要是 consul 和 etcd。
*   弱一致性：如果对一致性要求不高，可以选择以 DNS 为基础的方案，也可以像新浪微博的 Vintage 一样基于 Redis 。

强一致性方案：

*   Zookeeper：来自Apache，强一致性（CP），使用Zab协议（基于PAXOS），Java编写。
*   Doozer：doozerd的Go语言客户端，强一致性，使用PAXOS协议。很多年前就存在的项目，已经停滞很久（没有特殊理由，不要选择），Go语言编写。
*   Etcd：据说是受zookeeper的Doozer启发，使用Raft协议，Go语言编写。
*   Consul：来自hashicorp公司，和etcd一样也是基于Raft协议，Consul最大的优势，是提供可以直接使用的成品，Go语言编写。
*   SmartStack：来自Airbnb，由Nerve和Synapse两个部分组成，依赖zookeeper和haproxy，Ruby语言编写。
*   Eureka：来自Netflix，服务器和客户端都是用Java语言编写，因此只能用于Java和基于JVM的语言，Java语言编写。
*   Serf：采用的是基于gossip的SWIM协议，Go语言编写。

弱一致性方案：

*   Vintage：新浪微博内部使用（没有见到开源，可以借鉴思路），基于Redis的轻量级KV存储系统。
*   Spotify(DNS)：基于DNS实现。
*   SkyDNS：基于DNS实现。
*   DNS by Consul：可以和Nginx集成，基于DNS实现。

内建服务注册：服务注册是任何一个服务化/微服务框架的必不可少的部分，很多框架内建了对服务注册的支持。三大主流方案：

*   Zookeeper
*   Etcd
*   Consul

负载均衡：

*   软件：Nginx、HA proxy、Apache、LVS
*   硬件：F5
*   实现方式：服务器端负载均衡、客户端负载均衡
*   单独使用：Netflix Ribbon
*   框架集成：所有框架都内置负载均衡的实现
*   最新消息：Github即将开源他们新设计的负载均衡系统GLB（Github Load Balancer），值得期待

熔断器目前只有一个可选的开源方案：Netflix Hystrix

# 第二部分：微服务框架

Dubbo：分布式服务框架，极富研究价值，但已死

Motan：新浪微博2016年8月开源，轻量的RPC框架，方便做一些适合自己业务场景的改造和功能feature，以达到内部业务平滑改造和迁移的目的。

Motan技术栈：

*   底层通讯引擎采用了Netty网络框架
*   序列化协议支持Hessian2和Java序列化
*   通讯协议支持Motan、HTTP、TCP、memcached
*   服务注册支持zookeeper和consul

Motan总结：

*   潜质：年轻，更新积极，有极大的发展潜力
*   现状：刚出来，功能和稳定性有待观察
*   使用：轻量级，入门门槛比dubbo低，但是特性少很多
*   局限：对跨语言调用支持较差，主要支持java

