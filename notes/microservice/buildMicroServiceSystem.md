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

Netflix OSS：微服务演进的典范

Netflix OSS组件拆分的比较细致，每个独立功能都拆分为单独的组件，方便按需选择：

*   Eureka：服务注册和服务发现
*   Archaius：分布式配置管理
*   Ribbon：客户端负载均衡
*   Hystrix：熔断器
*   Prana：Sidecar，帮助集成
*   Zuul：云网关gateway

Netflix OSS总结：

*   品质：非常高，可以直接用，也可以造轮子时做参考
*   社区：成熟，使用者很多，文档和资料齐全
*   口碑：有分歧，有人特别推崇，有人觉得坑多

作者观点（局限）：

*   先天不足：问世太早，当时只有AWS可选
*   权衡背景：如果不是直接使用AWS EC2......
*   当前选择：docker, mesos/kubernetes

根本原因：

*   OSS是5、6年前Netflix解决问题的思路和产出
*   而那时没有docker，没有微服务，没有mesos、kubermetes
*   如果现在重新设计，OSS的技术栈必然不同

建议：

*   不放弃，不盲目
*   谨慎选择，按需选择
*   避免不必要的复杂度

2014年spring boot的问世，才是最近三五年间spring最大的变革和重新思考。

Spring Boot定位：

*   致力于蓬勃发展的快速应用开发领域
*   用来简化新Spring应用的初始搭建以及开发过程
*   目标不在于为已解决的问题域提供新的解决方案
*   而是为平台带来新的开发体验
*   并简化对已有技术的使用

Spring Boot功能：

*   创建独立Spring应用
*   直接内嵌Tomcat, Jetty或者Undertow
*   提供'starter' POM文件来简化maven配置
*   尽可能的自动配置Spring
*   提供产品级别的功能如metrics，健康检查和外部化配置
*   完全没有代码生成并不需要XML配置

Spring Boot总结：

*   不是微服务框架
*   超广泛的群众基础
*   更适合作为微服务框架的基石
*   以Spring Boot为基础：Pivotal团队推出解决方案、Spring Cloud

Spring Cloud：基于Spring Boot的云应用开发工具，承载着spring对微服务架构领域的众望和抱负。

Spring Cloud定位：

*   目标：占领企业微服务架构
*   帮助开发完整的微服务系统
*   弥补Spring Boot的不足（或者说补充定位）
*   完全基于Spring Boot

Spring Cloud子项目：

*   Spring Cloud Config：中心化的外部配置管理，由git仓库支持
*   Spring Cloud Netflix：和多个Netflix OSS组件集成（Eureka, Hystrix, Zuul, Archaius, etc.）
*   Spring Cloud Bus：将服务和服务实例连接到分布式消息的event bus。用于在集群内传播状态变更。（如配置变更事件）
*   Spring Cloud Cluster：leader选举和通用有状态模式
*   Spring Cloud consul：使用Hashicorp公司的Consul来进行服务发现和配置管理
*   Spring Cloud Zookeeper：使用Apache Zookeeper做服务发现和配置管理
*   Spring Cloud Stream：消息中间件抽象层，目前支持Redis, Rabbit MQ和Kalfka
*   Spring Cloud Sleuth：用于Spring Cloud应用的分布式追踪，兼容Ziplin, HTrace和基于log（如ELK）的追踪

Spring Cloud总结：

*   有潜力：很年轻的项目，可以关注，前景看好
*   社区：最近使用的人越来越多，多次看到分享
*   建议：可以尝试，一般也能用，也可以考虑Spring Cloud Netflix套件

Spring Cloud Netflix：强强联手

作者对spring cloud的个人评价：想法很好，出发点正确，市场空缺而切入的时机很合适。但是，spring cloud的实际表现，总给人一种束手束脚，瞻前顾后，小富即安的感觉。对比十几年前 Rod Johnson 大叔意气风发，气壮山河，谈笑间掀翻EJB的王座的表现，如今的spring cloud，能力不足，信心不够，格局太小，难成大器。期待后面能有转变。

微服务框架的遗憾（一家之言）：

*   虽说选择有不少，真正满意的，**没有**！
*   期待未来，能有真正压倒性优势的强力产品面世

# 第三部分：基础设施

*   分布式配置管理
*   APM应用性能监控
*   日志分析
*   服务监控/告警

分布式配置管理存储方案：zookeeper、consul、etcd、Redis、数据库、Git仓库

分布式配置管理开源产品：

*   Disconf：来自百度，只支持Java，Zookeeper存储，Java语言编写
*   Qconf：来自奇虎360，支持c/c++、shell、php、python、lua、java、go、node等语言，zookeeper存储，C++语言编写
*   Diamond：来自阿里，淘宝网Java中间件团队的核心产品之一，不建议使用，mysql存储
*   Archaius：来自Netflix，Netflix OSS套件之一，Java语言编写

APM/应用性能监控领域的选择，商业产品很多，但是开源的选择不多，如下：

*   PinPoint：来自韩国团队，目前只支持Java，基于Dapper论文，Hbase存储
*   zipkin：Twitter开源，基于Dapper论文，mysql/Cassandra/ElasticSearch存储
*   CAT：大众点评开源，需要硬编码做"埋点"
*   HTrace：Clodera开源，捐献给Apache，还在孵化中

日志分析领域：ELK是王者，但是也有新秀出场：

*   ELK：Elasticsrach+Logstash+Kibana：开源实时日志分析平台；应用广泛，社区成熟，文档齐全
*   Prometheus+Grafana：新秀，参考Google Borgmon的设计；和ELK相比较轻，容易扩展；Cloud Native Computing Foundation的正式组件

# 结束语

*   仰望星空，看弱水三千：眼界要开阔，知识面要广，哪怕只是精通各种名字，至少，知道在某个地方有个好东西，知道某个领域有其他的选择
*   立足当下，吾只取一瓢：最终还是要落地的，能玩的转的东西才是好东西。另外，好东西虽多，找到一个能适合自己，能解决问题的就好了，别贪心，别贪多