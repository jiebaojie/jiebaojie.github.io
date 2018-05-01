---
layout: post
notes: true
subtitle: "【开源代码】PINPOINT"
comments: false
author: "NAVER"
date: 2018-05-01 00:00:00

---

GitHub地址：[https://github.com/naver/pinpoint/](https://github.com/naver/pinpoint/)


*   目录
{:toc }

# Pinpoint技术概述

注：内容翻译自官方文档 [Technical Overview Of Pinpoint](https://github.com/naver/pinpoint/wiki/Technical-Overview-Of-Pinpoint), 内容有点长，但是强烈推荐阅读！基本上这是目前pinpoint唯一的一份详细介绍设计和实现的资料。	

Pinpoint是一个分析大型分布式系统的平台，提供解决方案来处理海量跟踪数据。2012年七月开始开发，2015年1月9日作为开源项目启动。

本文将介绍Pinpoint： 什么促使我们开始搭建它， 用了什么技术， 还有Pinpoint agent是如何优化的。

## 开始动机 & Pinpoint特点

和如今相比， 过去的因特网的用户数量相对较小，而因特网服务的架构也没那么复杂。web服务通常使用两层(web 服务器和数据库)或三层（web服务器，应用服务器和数据库）架构。然而在如今，随着互联网的成长，需要支持大量的并发连接，并且需要将功能和服务有机结合，导致更加复杂的软件栈组合。更确切地说，比三层层次更多的n层架构变得更加普遍。SOA或者微服务架构成为现实。

系统的复杂度因此提升。系统越复杂，越难解决问题，例如系统失败或者性能问题。在三层架构中找到解决方案还不是太难，仅仅需要分析3个组件比如web服务器，应用服务器和数据库，而服务器数量也不多。但是，如果问题发生在n层架构中，就需要调查大量的组件和服务器。另一个问题是仅仅分析单个组件很难看到大局;当发生一个低可见度的问题时，系统复杂度越高，就需要更长的时间来查找原因。最糟糕的是，某些情况下我们甚至可能无法查找出来。

这样的问题也发生在NAVER的系统中。使用了大量工具如应用性能管理(APM)但是还不足以有效处理问题。因此我们最终决定为n层架构开发新的跟踪平台，为n层架构的系统提供解决方案。

Pinpoint的特点如下：

*	分布式事务跟踪，跟踪跨分布式应用的消息
*	自动检测应用拓扑，帮助你搞清楚应用的架构
*	水平扩展以便支持大规模服务器集群
*	提供代码级别的可见性以便轻松定位失败点和瓶颈
*	使用字节码增强技术，添加新功能而无需修改代码

本文将讲述Pinpoint的技术，例如事务跟踪和字节码增强。还会解释应用在pinpoint agent中的优化方法，agent修改字节码并记录性能数据。

## 分布式事务跟踪，基于google Dapper

pinpoint跟踪单个事务中的分布式请求，基于google Dapper。

### 在Google Dapper中分布式事务追踪是如何工作的

当一个消息从Node1发送到Node2(见下图)时，分布式追踪系统的核心是在分布式系统中识别在Node1中处理的消息和在Node2中出的消息之间的关系。

![](/img/notes/opensource/naverPinpoint/rpc.png)

问题在于无法在消息之间识别关系。例如，我们无法识别从Node1发送的第N个消息和Node2接收到的N'消息之间的关系。换句话说，当Node1发送完第X个消息时，是无法在Node2接收到的N的消息里面识别出第X个消息的。有一种方式试图在TCP或者操作系统层面追踪消息。但是，实现很复杂而且性能低下，而且需要为每个协议单独实现。另外，很难精确追踪消息。

不过，Google dapper实现了一个简单的解决方案来解决这个问题。这个解决方案通过在发送消息时添加应用级别的标签作为消息之间的关联。例如，在HTTP请求中的HTTP header中为消息添加一个标签信息并使用这个标签跟踪消息。

Pinpoint基于google dapper的跟踪技术,但是已经修改为在调用的header中添加应用级别标签数据以便在远程调用中跟踪分布式事务。标签数据由多个key组成，定义为TraceId。

## Pinpoint中的数据结构

Pinpoint中，核心数据结构由Span, Trace, 和 TraceId组成。

*	Span: RPC (远程过程调用/remote procedure call)跟踪的基本单元; 当一个RPC调用到达时指示工作已经处理完成并包含跟踪数据。为了确保代码级别的可见性，Span拥有带SpanEvent标签的子结构作为数据结构。每个Span包含一个TraceId。
*	Trace: 多个Span的集合; 由关联的RPC (Spans)组成. 在同一个trace中的span共享相同的TransactionId。Trace通过SpanId和ParentSpanId整理为继承树结构.
*	TraceId: 由 TransactionId, SpanId, 和 ParentSpanId 组成的key的集合. TransactionId 指明消息ID，而SpanId 和 ParentSpanId 表示RPC的父-子关系。
	*	TransactionId (TxId): 在分布式系统间单个事务发送/接收的消息的ID; 必须跨整个服务器集群做到全局唯一。
	*	SpanId: 当收到RPC消息时处理的工作的ID; 在RPC请求到达节点时生成。
	*	ParentSpanId (pSpanId): 发起RPC调用的父span的SpanId. 如果节点是事务的起点，这里将没有父span - 对于这种情况， 使用值-1来表示这个span是事务的根span。
	
Pinpoint中的术语"TransactionId"和google dapper中的术语"TraceId"有相同的含义。而Pinpoint中的术语"TraceId"引用到多个key的集合。

### TraceId如何工作

下图描述TraceId的行为，在4个节点之间执行了3次的RPC调用：

![](/img/notes/opensource/naverPinpoint/traceId.png)

在上图中，TransactionId (TxId) 体现了三次不同的RPC作为单个事务被相互关联。但是，TransactionId 本身不能精确描述PRC之间的关系。为了识别PRC之间的关系，需要SpanId 和 ParentSpanId (pSpanId). 假设一个节点是Tomcat，可以将SpanId想象为处理HTTP请求的线程，ParentSpanId代表发起这个RPC调用的SpanId。

使用TransactionId，Pinpoint可以发现关联的n个Span，并使用SpanId和ParentSpanId将这n个span排列为继承树结构。

SpanId 和 ParentSpanId 是 64位长度的整型。可能发生冲突，因为这个数字是任意生成的，但是考虑到值的范围可以从-9223372036854775808到9223372036854775807，不太可能发生冲突. 如果key之间出现冲突，Pinpoint和Google Dapper系统，会让开发人员知道发生了什么，而不是解决冲突。

TransactionId 由 AgentIds, JVM (java虚拟机)启动时间, 和 SequenceNumbers/序列号组成。

*	AgentId: 当Jvm启动时用户创建的ID; 必须在pinpoinit安装的全部服务器集群中全局唯一. 最简单的让它保持唯一的方法是使用hostname($HOSTNAME)，因为hostname一般不会重复. 如果需要在服务器集群中运行多个JVM，请在hostname前面增加一个前缀来避免重复。
*	JVM 启动时间: 需要用来保证从0开始的SequenceNumber的唯一性. 当用户错误的创建了重复的AgentId时这个值可以用来预防ID冲突。
*	SequenceNumber: Pinpoint agent 生成的ID, 从0开始连续自增;为每个消息生成一个.

Dapper 和 Zipkin, Twitter的一个分布式系统跟踪平台, 生成随机TraceIds (Pinpoint是TransactionIds) 并将冲突情况视为正常。然而, 在Pinpiont中我们想避免冲突的可能，因此实现了上面描述的系统。有两种选择：一是数据量小但是冲突的可能性高，二是数据量大但是冲突的可能性低。我们选择了第二种。

可能有更好的方式来处理transaction。我们起先有一个想法，通过中央key服务器来生成key。如果实现这个模式，可能导致性能问题和网络错误。因此，大量生成key被考虑作为备选。后面这个方法可能被开发。现在采用简单方法。在pinpoint中，TransactionId被当成可变数据来对待。

## 字节码增强，无需代码修改

前面我们解释了分布式事务跟踪。实现的方法之一是开发人员自己修改代码。当发生RPC调用时容许开发人员添加标签信息。但是，修改代码会成为包袱，即使这样的功能对开发人员非常有用。

Twitter的 Zipkin 使用修改过的类库和它自己的容器(Finagle)来提供分布式事务跟踪的功能。但是，它要求在需要时修改代码。我们期望功能可以不修改代码就工作并希望得到代码级别的可见性。为了解决这个问题，pinpoint中使用了字节码增强技术。Pinpoint agent干预发起RPC的代码以此来自动处理标签信息。

### 克服字节码增强的缺点

字节码增强在手工方法和自动方法两者之间属于自动方法。

*	手工方法：开发人员使用ponpoint提供的API在关键点开发记录数据的代码
*	自动方法：开发人员不需要代码改动，因为pinpoint决定了哪些API要调节和开发

下面是每个方法的优点和缺点：

| | 优点 | 缺点 |
| - | - | - |
| 手工跟踪 | 1. 要求更少开发资源 2. API可以更简单并最终减少bug的数量 | 1. 开发人员必须修改代码 2. 跟踪级别低 |
| 自动跟踪 | 1. 开发人员不需要修改代码 2. 可以收集到更多精确的数据因为有字节码中的更多信息 | 1. 在开发pinpoint时，和实现一个手工方法相比，需要10倍开销来实现一个自动方法 2. 需要更高能力的开发人员，可以立即识别需要跟踪的类库代码并决定跟踪点 3. 增加bug发生的可能性，因为使用了如字节码增强这样的高级开发技巧 |

字节码增强是一种高难度和高风险的技术。但是，综合考虑使用这种技术开所需要的资源和难度，使用它仍然有很多的益处。

### 字节码增强的价值

我们选择字节码增强的理由，除了前面描述的那些外，还有下面的强有力的观点：

#### 隐藏API

一旦API被暴露给开发人员使用，我们作为API的提供者，就不能随意的修改API。这样的限制会给我们增加压力。

我们可能修改API来纠正错误设计或者添加新的功能。但是，如果做这些受到限制，对我们来说很难改进API。解决这个问题的最好的答案是一个可升级的系统设计，而每个人都知道这不是一个容易的选择。如果我们不能掌控未来，就不可能创建完美的API设计。

而使用字节码增强技术，我们就不必担心暴露跟踪API而可以持续改进设计，不用考虑依赖关系。对于那些计划使用pinpoint开发应用的人，换一句话说，这代表对于pinpoint开发人员，API是可变的。现在，我们将保留隐藏API的想法，因为改进性能和设计是我们的第一优先级。

### 容易启用或者禁用

使用字节码增强的缺点是当Pinpoint自身类库的采样代码出现问题时可能影响应用。不过，可以通过启用或者禁用pinpoint来解决问题，很简单，因为不需要修改代码。

通过增加下面三行到JVM启动脚本中就可以轻易的为应用启用pinpoint：

	-javaagent:$AGENT_PATH/pinpoint-bootstrap-$VERSION.jar
	-Dpinpoint.agentId=<Agent's UniqueId>
	-Dpinpoint.applicationName=<The name indicating a same service (AgentId collection)>
	
如果因为pinpoint发生问题，只需要在JVM启动脚本中删除这些配置数据。