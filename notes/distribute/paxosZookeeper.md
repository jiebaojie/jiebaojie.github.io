---
layout: post
notes: true
subtitle: "从Paxos到ZooKeeper（分布式一致性原理与实践）"
comments: false
author: "倪超"
date: 2016-12-24 00:00:00

---

![](/img/notes/distribute/paxosZookeeper/paxos_zookeeper.jpg)

*   目录
{:toc }

# 问题的提出

在计算机科学领域，分布式一致性问题是一个相当重要，且被广泛探索与论证的问题，通常存在于诸如分布式文件系统、缓存系统和数据库等大型分布式存储系统中。

## 终端用户

*   火车站售票：严格的一致性要求——系统的数据，无论在哪个售票窗口，每时每刻都必须是准确无误的。
*   银行转账：需要为用户保证绝对可靠的数据安全，虽然在数据一致性上存在延时，但最终务必保证严格的一致。
*   网上购物：虽然向用户展示了一些可以说是“错误”的数据，但是在整个系统使用过程中，一定会在某一个流程上对系统数据进行准确无误的检查，从而避免用户发生不必要的损失。

我们的终端用户在使用不同的计算机产品时对于数据一致性的需求是不一样的。

## 更新的并发性

《深入理解计算机系统》一书对并发的定义：如果逻辑控制流在时间上重叠，那么它们就是并发的。

## 分布式一致性问题

分布式系统对于数据的复制需求一般都来自于以下两个原因：

*   为了提高系统的可用性，以防止单点故障引起的系统不可用。
*   提升系统的整体性能，通过负载均衡技术，能够让分布在不同地方的数据副本都能够为用户提供服务。

分布式一致性问题：在分布式环境中引入数据复制机制后，不同数据节点间可能出现的，并无法依靠计算机应用程序自身解决的数据不一致的情况。

一致性级别：

*   强一致性
*   弱一致性
    *   会话一致性：对于写入的值，在同一个客户端会话中可以立即读到一致的值，但其他的会话不能保证。
    *   用户一致性：对于写入的值，在同一个用户中可以立即读到一致的值，但其他用户不能保证。
*   最终一致性：弱一致性的一个特例，系统会保证在一定时间内，能够达到一个数据一致的状态。

# 第1章 分布式架构

## 1.1 从集中式到分布式

### 1.1.1 集中式的特点

集中式系统的最大的特点就是部署结构简单。

### 1.1.2分布式的特点

《分布式系统概念与设计》定义：分布式系统是一个硬件或软件组件分布在不同的网络计算机上，彼此之间仅仅通过消息传递进行通信和协调的系统。

一个标准的分布式系统在没有任何特定业务逻辑约束的情况下，都会有如下几个特征：

*   分布性
*   对等性
*   并发性
*   缺乏全局时钟
*   故障总是会发生

### 1.1.3 分布式环境的各种问题

#### 通信异常

从集中式向分布式演变的过程中，必然引入了网络因素，而由于网络本身的不可靠性，因此也引入了额外的问题。

#### 网络分区

当网络由于发生异常情况，导致分布式系统中部分节点之间的网络延时不断增大，最终导致组成分布式系统的所有节点中，只有部分节点之间能够进行正常通信，而另一些节点则不能——我们将这个现象称为网络分区，就是俗称的“脑裂”。

#### 三态

成功、失败、超时

超时通常的两种情况：

*   由于网络原因，该请求（消息）并没有被成功地发送到接收方，而是在发送过程就发生了消息丢失现象。
*   该请求（消息）成功的被接收方接收后，并进行了处理，但是在将响应反馈给发送方的过程中，发生了消息丢失现象。

#### 节点故障

组成分布式系统的服务器节点出现宕机或“僵死”现象。通常根据经验来说，每个节点都有可能出现故障，并且每天都在发生。

## 1.2 从ACID到CAP/BASE

### 1.2.1 ACID

#### 原子性

事务的原子性是指事务必须是一个原子的操作序列单元。事务中包含的各项操作在一起执行的过程中，只允许出现以下两种状态之一：

*   全部成功执行
*   全部不执行

#### 一致性

事务的执行结果必须是使数据库从一个一致性状态转变到另一个一致性状态，因此当数据库只包含成功事务提交的结果时，就能说数据库处于一致性状态。

#### 隔离性

事务的隔离性是指在并发环境中，并发的事务是相互隔离的，一个事务的执行不能被其他事务干扰。

4个事务隔离级别：

*   未授权读取：也被称为读未提交（Read Uncommited），该隔离级别允许脏读取，其隔离级别最低。
*   授权读取：也被称为读已提交（Read Commited），只允许获取已经被提交的数据。
*   可重复读取：保证在事务处理过程中，多次读取同一个数据时，其值都和事务开始时刻是一致的。因此该事务级别禁止了不可重复读取和脏读取，其值都和事务开始时刻是一致的。
*   串行化：最严格的事务隔离级别。它要求所有事务都被串行执行，即事务只能一个接一个地进行处理，不能并发执行。

事务隔离级别越高，就越能保证数据的完整性和一致性，但同时对并发性能的影响也越大。

#### 持久性

是指一个事务一旦提交，它对数据库中对应数据的状态变更就应该是永久性的。

### 1.2.2 分布式事务

分布式事务是指事务的参与者、支持事务的服务器、资源服务器以及事务管理器分别位于分布式系统的不同节点之上。通常一个分布式事务中会涉及对多个数据源或业务系统的操作。

由于在分布式事务中，各个子事务的执行是分布式的，因此要实现一种能保证ACID特性的分布式事务处理系统就显得格外复杂。

### 1.2.3 CAP和BASE理论

在可用性和一致性之间永远无法存在一个两全其美的方案，于是如何构建一个兼顾可用性和一致性的分布式系统成为了无数工程师探讨的难题，出现了诸如CAP和BASE这样的分布式系统经典理论。

#### CAP定理

CAP理论告诉我们，一个分布式系统不可能同时满足一致性（C：Consistency）、可用性（Availability）和分区容错性（P：Partition tolerance）这三个基本需求，最多只能满足其中的两项。

##### 一致性

在分布式环境中，一致性是指数据在多个副本之间是否能够保持一致的特性。

在分布式系统中，如果能够做到针对一个数据项的更新操作执行成功后，所有的用户都可以读取到其最新的值，那么这样的系统就被认为具有强一致性（或严格的一致性）。

##### 可用性

可用性是指系统提供的服务必须一直处于可用的状态，对于用户的每一个操作请求总是能够在**有限的时间内返回结果**。

##### 分区容错性

分布式系统在遇到任何网络分区故障的时候，仍然需要能够保证对外提供满足一致性和可用性的服务，除非是整个网络环境都发生了故障。

![](/img/notes/distribute/paxosZookeeper/cap.jpg)

CAP定理应用：

*   放弃P：如果希望能够避免系统出现分区容错性问题，一种较为简单的做法是将所有的数据（或者仅仅是那些与事务相关的数据）都放在一个分布式节点上。这样的做法虽然无法100%地保证系统不会出错，但至少不会碰到由于网络分区带来的负面影响。但同时需要注意的是，放弃P的同时也就意味着放弃了系统的可扩展性。
*   放弃A：放弃A的做法是一旦系统遇到网络分区或其他故障时，那么受到影响的服务需要等待一定的时间，因此在等待期间系统无法对外提供正常的服务，即不可用。
*   放弃C：放弃一致性指的是放弃数据的强一致性，而保留数据的最终一致性。这样的系统无法保证数据保持实时的一致性，但是能够承诺的是，数据最终会达到一个一致的状态。

系统架构设计师往往需要把精力花在如何根据业务特点在C（一致性）和A（可用性）之间寻求平衡。

#### BASE理论

BASE是Basically Available（基本可用）、Soft state（软状态）和Eventually consistent（最终一致性）三个短语的简写。BASE是对CAP中一致性和可用性权衡的结果，其来源于对大规模互联网系统分布式实践的总结，是基于CAP定理逐步演化而来的，其核心思想是即使无法做到强一致性（Strong consistency），但每个应用都可以根据自身的业务特点，采用适当的方式来使系统达到最终一致性（Eventual consistency）。

##### 基本可用

基本可用是指分布式系统在出现不可预知故障的时候，允许损失部分可用性。

*   响应时间上的损失（如查询结果的响应时间增加到1-2秒）
*   功能上的损失（降级）

##### 弱状态

弱状态也称为软状态，和硬件状态相对，是指允许系统中的数据存在中间状态，并认为该中间状态的存在不会影响系统的整体可用性，即允许系统在不同节点的数据副本之间进行数据同步的过程存在延时。

##### 最终一致性

最终一致性强调的是系统中所有的数据副本，在经过一段时间的同步后，最终能够达到一个一致的状态。

最终一致性是一种特殊的弱一致性：系统能够保证在没有其他新的更新操作的情况下，数据最终一定能够达到一致的状态，因此所有客户端对系统的数据访问都能够获取到最新的值。同时，在没有发生故障的前提下，数据达到一致性状态的时间延迟，取决于网络延迟、系统负载和数据复制方案设计等因素。

在实际工程实践中，最终一致性存才以下五类主要变种：

*   因果一致性（Causal consistency）：如果进程A在更新完某个数据项后通知了进程B，那么进程B之后对该数据的访问都应该能够获取到进程A更新后的最新值。
*   读己之写（Read your writes）：进程A更新一个数据项之后，它自己总是能够访问到更新过的最新值，而不会看到旧值。
*   会话一致性（Session Consistency）：系统保证在同一个有效的会话中实现“读己之所写”的一致性。
*   单调读一致性（Monotonic read consistency）：单调读一致性是指如果一个进程从系统中读取出一个数据项的某个值后，那么系统对于该进程后续任何数据访问都不应该返回更旧的值。
*   单调写一致性（Monotonic write consistency）：单调写一致性是指，一个系统需要能够保证来自同一个进程的写操作被顺序地执行。

BASE理论面向的是大型高可用可扩展的分布式系统，和传统事务的ACID特性是相反的，它完全不同于ACID的强一致性模型，而是提出通过牺牲强一致性来获得可用性，并允许数据在一段时间内是不一致的，但最终达到一致状态。

## 小结

计算机系统从集中式向分布式的变革伴随着包括分布式网络、分布式事务和分布式数据一致性等在内的一系列问题与挑战，同时也催生了一大批诸如ACID、CAP和BASE等经典理论的快速发展。

# 第2章 一致性协议

## 2.1 2PC与3PC

当一个事务操作需要跨越多个分布式节点的时候，为了保持事务处理的ACID特性，就需要引入一个称为“协调者”（Coodinator）的组件来统一调度所有分布式节点的执行逻辑，这些被调度的分布式节点则被称为“参与者”（Participant）。协调者负责调度参与者的行为，并最终决定这些参与者是否要把事务真正进行提交。

### 2.1.1 2PC

![](/img/notes/distribute/paxosZookeeper/2pc.jpg)

优点：原理简单，实现方便。

缺点：

*   同步阻塞
*   单点问题：协调者的角色在整个二阶段提交协议中起到了非常重要的作用。一旦协调者出现问题，那么整个二阶段提交流程将无法运转，更为严重的是，如果协调者是在阶段二中出现问题的话，那么其他参与者将会一直处于锁定事务资源的状态中，而无法继续完成事务操作。
*   数据不一致（脑裂）：在二阶段提交协议的阶段二，即执行事务提交的时候，当协调者向所有的参与者发送Commit请求之后，发生了局部网络异常或者是协调者在尚未发送完Commit请求之前自身发生了崩溃，导致最终只有部分参与者收到了Commit请求。
*   二阶段提交协议没有设计较为完善的容错机制，任意一个节点的失败都会导致整个事务的失败。

### 2.1.2 3PC

3PC，是Tree-Phase Commit的缩写，即三阶段提交，是2PC的改进版，其将二阶段提交协议的“提交事务请求”过程一分为二，形成了由CanCommit、PreCommit和doCommit三个阶段组成的事务处理协议。

优点：相较于二阶段提交协议，三阶段提交协议最大的优点就是降低了参与者的阻塞范围，并且能够在出现单点故障后继续达成一致。

缺点：三阶段提交协议在去除阻塞的同时也引入了新的问题，那就是在参与者接收到preCommit消息后，如果网络出现分区，此时协调者所在的节点和参与者无法进行正常的网络通信，在这种情况下，该参与者依然会进行事务的提交，这必然出现数据的不一致。

## 2.2 Paxos算法

Paxos算法是莱斯利兰伯特（Leslie Lamport）于1990年提出的一种基于消息传递且具有高度容错特性的一致性算法，是目前公认的解决分布式一致性问题最有效的算法之一。

Paxos算法需要解决的问题就是如何在一个可能发生上述异常的分布式系统中，快速且正确地在集群内部对某个数据的值达成一致，并且保证不论发生以上任何异常，都不会破坏整个系统的一致性。

### 2.2.1 追本溯源

### 2.2.2 Paxos理论的诞生

论文：The Part-Time Parliament

### 2.2.3 Paxos算法详解

#### 问题描述

假设有一组可以提出提案的进程集合，那么对于一个一致性算法来说需要保证以下几点：

*   在这些被提出的提案中，只有一个会被选定。
*   如果没有提案被提出，那么就不会有被选定的提案。
*   当一个提案被选定后，进程应该可以获取被选定的提案信息。

对于一致性来说，安全性（Safety）需求如下：

*   只有被提出的提案才能被选定（Chosen）。
*   只能有一个值被选定。
*   如果某个进程认为某个提案被选定了，那么这个提案必须是真的被选定的那个。

一致性算法中的三种参与角色：Proposer、Acceptor和Learner。在具体的实现中，一个进程可能充当不止一种角色。假设不同参与者之间可以通过收发消息来进行通信，那么：

*   每个参与者以任意的速度执行，可能会因为出错而停止，也可能会重启。
*   消息在传输过程中可能会出现不可预知的延迟，也可能会重复或丢失，但是消息是不会被损坏，即消息内容不会被篡改。

#### 提案的选定

我们假定足够多的Acceptor是整个Acceptor集合的一个子集，并且让这个集合大得可以包含Acceptor集合中的大多数成员，因为任意两个包含大多数Acceptor的子集至少有一个公共成员。另外我们再规定，每一个Acceptor最多只能批准一个提案，那么就能保证只有一个提案被选定了。

##### 推导过程

**P1：一个Acceptor必须批准它收到的第一个提案。**

问题：

*   不同的Proposer分别提出多个提案，无法选定一个提案。
*   每个提案都被差不多一半的Acceptor批准了，此时即使只有一个Acceptor出错，都可能导致无法确定该选定哪个提案。

![](/img/notes/distribute/paxosZookeeper/proposer_acceptor_different.jpg)

![](/img/notes/distribute/paxosZookeeper/one_acceptor_fail.jpg)

**P2：如果编号为M<sub>0</sub>，Value值为V<sub>0</sub>的提案（即[M<sub>0</sub>, V<sub>0</sub>]）被选定了，那么所有比编号M<sub>0</sub>更高的，且被选定的提案，其Value值必须也是V<sub>0</sub>。**

**P2a：如果编号为M<sub>0</sub>，Value值为V<sub>0</sub>的提案（即[M<sub>0</sub>, V<sub>0</sub>]）被选定了，那么所有比编号M<sub>0</sub>更高的，且被Acceptor批准的提案，其Value值必须也是V<sub>0</sub>。**

![](/img/notes/distribute/paxosZookeeper/proposer_without_one_acceptor.jpg)

**P2b：如果一个提案[M<sub>0</sub>, V<sub>0</sub>]被选定后，那么之后任何Proposer产生的编号更高的提案，其Value值都为V<sub>0</sub>。**

**P2c：对于任意的M<sub>n</sub>和V<sub>n</sub>，如果提案[M<sub>n</sub>, V<sub>n</sub>]被提出，那么肯定存在一个由半数以上的Acceptor组成的集合S，满足以下两个条件中的任意一个：**

*   **S中不存在任何批准的编号小于M<sub>n</sub>的提案的Acceptor。**
*   **选取S中所有Acceptor批准的编号小于M<sub>n</sub>的提案，其中编号最大的那个提案其Value值是V<sub>n</sub>。**

实际上P2c规定了每个Proposer如何产生一个提案：对于产生的每个提案[M<sub>n</sub>, V<sub>n</sub>]，需要满足如下条件：

** 存在一个由超过半数的Acceptor组成的集合S：**

*   **要么S中没有Acceptor批准过编号小于M<sub>n</sub>的任何提案。**
*   **要么S中的所有Acceptor批准的所有编号小于M<sub>n</sub>的提案中，编号最大的那个提案的Value值为V<sub>n</sub>。**

##### Proposer生成提案

1.  Proposer选择一个新的提案编号M<sub>n</sub>，然后向某个Acceptor集合的成员发送请求，要求该集合中的Acceptor做出如下回应：
    *   向Propeser承诺，保证不再批准任何编号小于M<sub>n</sub>的提案。
    *   如果Acceptor已经批准过任何提案，那么其向Proposer反馈当前该Acceptor已经批准的编号小于M<sub>n</sub>但为最大编号的那个提案的值。

    *我们将该请求称为编号为M<sub>n</sub>的提案的Prepare请求。*

2.  如果Proposer收到了来自半数以上的Acceptor的响应结果，那么它就可以产生编号为M<sub>n</sub>、Value值为V<sub>n</sub>的提案，这里的V<sub>n</sub>是所有响应中编号最大的提案的Value值。当然还存在另一种情况，就是半数以上的Acceptor都没有批准过任何提案，即响应中不包含任何的提案，那么此时V<sub>n</sub>值就可以由Proposer任意选择。
    
    *在确定提案之后，Proposer就会将该提案再次发送给某个Acceptor集合，并期望获得它们的批准，我们称此请求为Accept请求。*

##### Acceptor批准提案

一个Acceptor可能会收到来自Proposer的两种请求，分别是Prepare请求和Accept请求，对这两类请求做出响应的条件分别如下：

*   Prepare请求：Acceptor可以在任何时候响应一个Prepare请求。
*   Accept请求：在不违背Accept现有承诺的前提下，可以任意响应Accept请求。因此，对Acceptor逻辑处理的约束条件，大体可以定义如下：

**P1a：一个Acceptor只要尚未响应过任何编号大于M<sub>n</sub>的Prepare请求，那么它就可以接受这个编号为M<sub>n</sub>的提案。

##### 算法优化

假设一个Acceptor收到了一个编号为M<sub>n</sub>的Prepare请求，但此时该Acceptor已经对编号大于M<sub>n</sub>的Prepare请求做出了响应，因此它肯定不会再批准任何新的编号为M<sub>n</sub>的提案，那么很显然，Acceptor就没有必要对这个Prepare请求做出响应，于是Acceptor可以选择忽略这样的Prepare请求。同时，Acceptor也可以忽略掉那些它已经批准过的提案的Prepare请求。

##### 算法陈述

*   阶段一
    1.  Proposer选择一个提案编号M<sub>n</sub>，然后向Acceptor的某个超过半数的子集成员发送编号为M<sub>n</sub>的Prepare请求。
    2.  如果一个Acceptor收到一个编号为M<sub>n</sub>的Prepare请求，且编号M<sub>n</sub>大于该Acceptor已经响应的所有Prepare请求的编号，那么它就会将它已经批准过的最大编号的提案作为响应反馈给Proposer，同时该Acceptor会承诺不会再批准任何编号小于M<sub>n</sub>的提案。
*   阶段二
    1.  如果Proposer收到来自半数以上的Acceptor对于其发出的编号为M<sub>n</sub>的Prepare请求的响应，那么它就会发送一个针对[M<sub>n</sub>, V<sub>n</sub>]提案的Accept请求给Acceptor。注意，V<sub>n</sub>的值就是收到的响应中编号最大的提案的值，如果响应中不包含任何提案，那么它就是任意值。
    2.  如果Acceptor收到这个针对[M<sub>n</sub>, V<sub>n</sub>]提案的Accept请求，只要该Acceptor尚未对编号大于M<sub>n</sub>的Prepare请求做出响应，它就可以通过这个提案。

#### 提案的获取

##### 方案一

Learner获取一个已经被选定的提案的前提是，该提案已经被半数以上的Acceptor批准。因此，最简单的做法就是一旦Acceptor批准了一个提案，就将该提案发送给所有的Learner。

很显然，这种做法虽然可以Learner尽快地获取被选定的提案，但是却需要让每个Acceptor与所有的Learner逐个进行一次通信，通信的次数至少为两者个数的乘积。

##### 方案二

我们可以让所有的Acceptor将它们对提案的批准情况，统一发送给一个特定的Learner（主Learner）。当主Learner被通知一个提案已经被选定时，它会负责通知其他的Learner。

较方案一而言，方案二虽然需要多一个步骤才能将提案通知到所有的Learnder，但其通信次数却大大减少了，通常只是Acceptor和Learner的个数总和。但同时，该方案引入了一个新的不稳定因素：主Learner随时可能出现故障。

##### 方案三

可以将主Learner的范围扩大，即Acceptor可以将批准的提案发送给一个特定的Learner集合，该集合中的每个Learner都可以在一个提案被选定后通知所有其他的Learner。这个Learner集合中的Learner个数越多，可靠性就越好，但同时网络通信的复杂度也越高。

#### 通过选取主Proposer保证算法的活性

![](/img/notes/distribute/paxosZookeeper/two_proposer_dead_cicle.jpg)

为了保证Paxos算法流程的可持续性，以避免陷入“死循环”，就必须选择一个主Proposer，并规定只有主Proposer才能提出议案。

## 小结

Paxos算法引入“过半”的理念，通俗地讲就是少数服从多数的原则。同时，Paxos算法支持分布式节点角色之间的轮换，这极大地避免了分布式单点的出现，因此Paxos算法既解决了无限期待问题，也解决了“脑裂”问题，是目前来说最优秀的分布式一致性协议之一。

# 第3章 Paxos的工程实践

## 3.1 Chubby

Google Chubby是一个大名鼎鼎的分布式服务，GFS和Big Table等大型系统都用它来解决分布式协作、元数据存储和Master选举等一系列与分布式锁服务相关的问题。Chubby的底层一致性实现就是以Paxos算法为基础的，这给Paxos算法的学习者提供了一个理论联系的范例，从而可以了解到Paxos算法是如何在实际工程中得到应用的。

### 3.1.1 概述

Chubby是一个面向松耦合分布式系统的锁服务，通常用于为一个由大量小型计算机构成的松耦合分布式系统提供高可用的分布式锁服务。一个分布式锁服务的目的是允许它的客户端进程同步彼此的操作，并对当前所处环境的基本状态信息达成一致。针对这个目的，Chubby提供了粗粒度的分布式锁服务，开发人员不需要使用复杂的同步协议，而是直接调用Chubby的锁服务接口即可实现分布式系统中多个进程之间粗粒度的同步控制，从而保证分布式数据的一致性。

Chubby的客户端接口设计非常类似于UNIX文件系统结构，应用程序通过Chubby的客户端接口，不仅能够对Chubby服务器上的整个文件进行读写操作，还能够添加对文件节点的锁控制，并且能够订阅Chubby服务端发出的一系列文件变动的事件通知。

### 3.1.2 应用场景

在Chubby的众多应用场景中，最为典型的就是集群中服务器的Master选举：

*   Google文件系统（Google File System, GFS）中使用Chubby锁服务来实现对GFS Master服务器的选举
*   BigTable中，Chubby同样被用于Master选举，并且借助Chubby，Master能够非常方便地感知其所控制的那些服务器。同时，通过Chubby，BigTable的客户端还能够方便地定位到当前Bigtable集群的Master服务器。

此外，在GFS和Bigtable中，都使用Chubby来进行系统运行时元数据的存储。

### 3.1.3 设计目标

将Chubby设计成一个需要访问中心化节点的分布式锁服务。

锁服务具有以下4个传统算法库所不具有的优点：

*   对上层应用程序的侵入性更小：与通过一个封装了分布式一致性协议的客户端库相比，使用一个分布式锁服务的接口方式对上层应用程序的侵入性会更小，并且更易于保持系统已有的程序结构和网络通信模式。
*   便于提供数据的发布与订阅：一方面能够大大减少客户端依赖的外部服务，另一方面，数据的发布与订阅功能和锁服务在分布式一致性特性上是相通的。
*   开发人员对基于锁的接口更为熟悉
*   更便捷地构建更可靠的服务

在Chubby的设计过程中，提出了以下几个设计目标：

*   提供一个完整的、独立的分布式锁服务，而非仅仅是一个一致性协议的客户端库
*   提供粗粒度的锁服务：当锁服务短暂失效时（例如服务器宕机），Chubby需要保持所有锁的持有状态，以避免持有锁的客户端出现问题。这和细粒度锁的设计方式有很大的区别，细粒度通常设计为锁服务一旦失效就释放所有锁，因为细粒度锁的持有时间很短，相比而言放弃锁带来的代价较小。
*   在提供锁服务的同时提供对小文件的读写功能：Chubby提供对小文件的读写服务，以使得被选举出来的Master可以在不依赖额外服务的情况下，非常方便地向所有客户端发布自己的状态信息。
*   高可用、高可靠：在Chubby的实际应用过程中，必须能够支撑成百上千个Chubby客户端对同一个文件进行监视和读取。
*   

### 3.1.4 Chubby技术架构

#### 系统结构

Chubby的整个系统结构主要由服务端和客户端两部分组成，客户端通过RPC调用与服务器进行通信。

![](/img/notes/distribute/paxosZookeeper/chubby_server_client.jpg)

#### 目录与文件

Chubby对外提供了一套与Unix文件系统非常相近但是更简单的访问接口。典型的节点路径表示如下：

    /ls/foo/wombat/pouch

其中，ls是所有Chubby节点所共有的前缀，代表着锁服务，是Lock Service的缩写；foo则指定了Chubby集群的名字，从DNS可以查询到由一个或多个服务器组成该Chubby集群；剩余部分的路径/wombat/pouch则是一个真正包含业务含义的节点名字，由Chubby服务器内部解析并定位到数据节点。

Chubby的客户端应用程序可以通过自定义的文件系统访问接口来访问Chubby服务端数据，比如可以使用GFS的文件系统访问接口，这就大大减少了用户使用Chubby的成本。

Chubby上的每个数据节点都分为持久节点和临时节点两大类，其中持久节点需要显式地调用接口API来进行删除，而临时节点则会在其对应的客户端会话失效后被自动删除。

另外，Chubby上的每个数据节点都包含了少量的元数据信息，其中包括用于权限控制的访问控制列表（ACL）信息。同时，每个节点的元数据中还包括4个单调递增的64位编号，分别如下：

*   实例编号：实例编号用于标识Chubby创建该数据节点的顺序，节点的创建顺序不同，其实例编号也不同（客户端可方便地识别出同名节点是否是同一个数据节点）。
*   文件内容编号（只针对文件）：文件内容编号用于标识文件内容的变化情况，该编号会在文件内容被写入时增加。
*   锁编号：锁编号用于标识节点锁状态变更情况，该编号会在节点锁从自由（free）状态转换到被持有（held）状态时增加。
*   ACL编号：ACL编号用于标识节点的ACL信息变更情况，该编号会在节点的ACL配置信息被写入时增加。

同时，Chubby还会标识一个64位的文件内容校验码，以便客户端能够识别出文件是否变更。

#### 锁与锁序列器

在Chubby中，任意一个数据节点都可以充当一个读写锁来使用：一种是单个客户端以排他（写）模式持有这个锁，另一种是任意数目的客户端以共享（读）模式持有这个锁。

同时，在Chubby的锁机制中需要注意的是，Chubby舍弃了严格的强制锁，客户端可以在没有获取任何锁的情况下访问Chubby的文件，也就是说，持有锁F既不是访问文件F的必要条件，也不会阻止其他客户端访问文件F。

在Chubby中，主要采用锁延迟和锁序列器两种策略来解决由于消息延迟和重排序引起的分布式锁问题。

*   锁延迟：一种简单的策略，使用Chubby的应用几乎不需要进行任何的代码修改。具体的，如果一个客户端以正常的方式主动释放了一个锁，那么Chubby服务器将会允许其他客户端能够立即获取到该锁。而如果一个锁是因为客户端的异常情况（如客户端无响应）而被释放的话，那么Chubby服务器会为该锁保留一定的时间，我们称之为“锁延迟”（lock-delay），在这段时间内，其他客户端无法获取这个锁。
*   锁序列器：该策略需要Chubby的上层应用配合在代码中加入响应的修改逻辑。任何时候，锁的持有者都可以向Chubby请求一个锁序列器，其包括锁的名字、锁模式（排他或共享模式），以及锁序列号。当客户端应用程序进行一些需要锁机制保护的操作时，可以将该锁序列器一并发送给服务端。Chubby服务端接收到这样的请求后，会首先检测该序列器是否有效，以及检查客户端是否处于恰当的锁模式，如果没有通过检查，那么服务端就会拒绝该客户端请求。

#### Chubby中的事件通知机制

Chubby的客户端可以向服务端注册事件通知，当触发这些事件的时候，服务端就会向客户端发送对应的事件通知。常见的Chubby事件如下：

*   文件内容变更
*   节点删除
*   子节点新增、删除
*   Master服务器转移

#### Chubby的缓存

Chubby在客户端中实现了缓存，会在客户端对文件内容和元数据信息进行缓存。

在Chubby中，通过租期机制来保证缓存的一致性。

*   每个客户端的缓存都有一个租期，一旦该租期到期，客户端就需要向服务器续订租期以继续维持缓存的有效性。
*   当文件数据或元数据信息被修改时，Chubby服务端首先会阻塞该修改操作，然后由Master向所有可能缓存了该数据的客户端发送缓存过期信号，以使其缓存失效，等到Master在接收到所有相关客户端针对该过期信号的应答（应答包括两类，一类是客户端明确要求更新缓存，另一类则是客户端允许缓存租期过期）后，再继续进行之前的修改操作。

Chubby的缓存数据保证了强一致性。

#### 会话和会话激活（KeepAlive）

Chubby客户端和服务端之间通过创建一个TCP连接来进行所有的网络通信操作，我们将这一连接称为会话（Session）。会话是有生命周期的，存在一个超时时间，在超时时间内，Chubby客户端和服务端之间可以通过心跳检测来保持会话的活性，以使会话周期得以延续，我们将这个过程称为KeepAlive（会话激活）。如果能够成功地通过KeepAlive过程将Chubby会话一直延续下去，那么客户端创建的句柄、锁和缓存数据等都依然有效。

#### KeepAlive请求

Master在接收到客户端的KeepAlive请求时，首先会将该请求阻塞住，并等到该客户端的当前会话租期即将过期时，才为其续租该客户端的会话租期，之后再向客户端响应这个KeepAlive请求。

客户端在接收到来自Master的续租响应后，会立即发起一个新的KeepAlive请求，再由Master进行阻塞。因此我们可以看出，每一个Chubby客户端总是会有一个KeepAlive请求阻塞在Master服务器上。

另外，Master还将通过KeepAlive响应来传递Chubby事件通知和缓存过期通知给客户端。

#### 会话超时

谈到会话租期，Chubby的客户端也会维持一个和Master端近似相同的会话租期。为什么是近似相同呢？这是因为客户端必须考虑两方面的因素：

*   一方面，KeepAlive响应在网络传输过程中会花费一定的时间
*   另一方面，Master服务端和Chubby客户端存在时钟不一致性现象。

危险状态：如果Chubby客户端在运行过程中，按照本地的会话租期超时时间，检测到其会话租期已经过期却尚未接收到Master的KeepAlive响应，那么这个时候，它将无法确定Master服务器是否已经中止了当前会话，我们称这个时候客户端处于“危险状态”。

此时，Chubby客户端会清空其本地缓存，并将其标记为不可用。同时，客户端还会等待一个被称作“宽限期”的时间周期（默认45秒）。如果在宽限期到期前，客户端和服务端之间成功地进行了KeepAlive，那么客户端就会再次开启本地缓存，否则，客户端就会认为当前会话已经过期了，从而中止本次会话。

当客户端进入上述提到的危险状态时，Chubby的客户端库会通过一个"jeopardy"事件来通知上层应用程序。如果恢复正常，客户端同样会以一个"safe"事件来通知应用程序可以继续正常运行了。但如果客户端最终没能从危险状态中恢复过来，那么客户端会以一个"expired"事件来通知应用程序当前Chubby会话已经超时。

#### Chubby Master故障恢复

Chubby的Master服务器上会运行着会话租期计时器，用来管理所有会话的生命周期。如果在运行过程中Master出现了故障，那么该计时器会停止，直到新的Master选举产生后，计时器才会继续计时，也就是说，从旧的Master崩溃到新的Master选举产生所花费的时间将不计入会话超时的计算中，这等价于延长了客户端的会话租期。如果新的Master在段时间内就选举产生了，那么客户端就可以在本地会话租期过期前与其创建连接。而如果Master的选举花费了较长的时间，就会导致客户端只能清空本地的缓存，并进入宽限期进行等待。

![](/img/notes/distribute/paxosZookeeper/chubby_master_recover.jpg)

一个新的Chubby Master服务器选举产生之后，会进行如下几个主要处理：

1.  新的Master选举产生后，首先需要确定Master周期。
2.  选举产生的新Master能够立即对客户端的Master寻址请求进行响应，但是不会立即开始处理客户端会话相关的请求操作。
3.  Master根据本地数据库中存储的会话和锁信息，来构建服务器的内存状态。
4.  到现在为止，Master已经能够处理客户端的KeepAlive请求了，但依然无法处理其他会话相关的操作。
5.  Master会发送一个"Master故障切换"事件给每一个会话，客户端接收到这个事件后，会清空它的本地缓存，并警告上层应用程序可能已经丢失了别的事件，之后再向Master反馈应答。
6.  此时，Master会一直等待客户端的应答，直到每一个会话都应答了这个切换事件。
7.  在Master接收到了所有客户端的应答之后，就能够开始所有的请求操作了。
8.  如果客户端使用了一个在故障切换之前创建的句柄，Master会重新为其创建这个句柄的内存对象，并执行调用。

### 3.1.5 Paxos协议实现

Chubby服务器的基本架构大致分为三层：

*   最底层是容错日志系统（Fault-Tolerant Log），通过Paxos算法能够保证集群中所有机器上的日志完全一致，同时具备较好的容错性。
*   日志层之上是Key-Value类型的容错数据库（Fault-Tolerant DB），其通过下层的日志来保证一致性和容错性。
*   存储层之上就是Chubby对外提供的分布式锁服务和小文件存储服务。

![](/img/notes/distribute/paxosZookeeper/chubby_structure.jpg)

## 3.2 Hypertable

Hypertable是一个使用C++语言开发的开源、高性能、可伸缩的数据库，其以Google的BigTable相关论文为基础指导，采用与HBase非常相似的分布式模型，其目的是要构建一个针对分布式海量数据的高并发数据库。

### 3.2.1 概述

目前Hypertable只支持最基本的添、删、改、查功能，对于事务处理和关联查询等关系型数据库的高级特性都尚未支持。同时，就少量数据记录的查询性能和吞吐量而言，Hypertable可能也不如传统的关系型数据库。和传统关系型数据库相比，Hypertable最大的优势在于以下几点：

*   支持对大量并发请求的处理。
*   支持对海量数据的管理。
*   扩展性良好，在保证可用性的前提下，能够通过随意添加集群中的机器来实现水平扩容。
*   可用性极高，具有非常号的容错性，任何节点的失效，既不会造成系统瘫痪也不会影响数据的完整性。

![](/img/notes/distribute/paxosZookeeper/hypertable_structure.png)

Hypertable的核心组件包括Hyperspace、RangeServer、Master和DFS Broker四部分。

*   Hyperspace是Hypertable中最重要的组件之一，其提供了对分布式锁服务的支持以及对元数据的管理，是保证Hypertable数据一致性的核心。
*   RangeServer是实际对外提供服务的组件单元，负责数据的读取和写入。在Hypertable中，通常会部署多个RangeServer，每个RangeServer都负责管理部分数据，由Master来负责进行RangeServer的集群管理。
*   Master是元数据管理中心，管理包括创建表、删除表或是其他表空间变更在内的所有元数据操作，同时负责检测RangeServer的工作状态，一旦某一个RangeServer宕机或是重启，能够自动进行Range的重新分配，从而实现对RangeServer集群的管理和负载均衡。
*   DFS Broker则是底层分布式文件系统的抽象层，用于衔接上层Hypertable和底层文件存储。所有对文件系统的读写操作，都是通过DFS Broker来完成的。

### 3.2.2 算法实现

#### Active Server

Hyperspace通常以一个服务器集群的形式部署，在运行过程中，会从集群中选举产生一个服务器作为Active Server，其余的服务器则是Standby Server。

#### 事务请求处理

在Hyperspace集群中，还有一个非常重要的组件，就是BDB。BDB服务也是采用集群部署的，也存在Master的角色，是Hyperspace底层实现分布式数据一致性的精华所在。

在Hyperspace对外提供服务时，任何对于元数据的操作，Master模块都会将其对应的事务请求发送给Hyperspace服务器。在接收到该事务请求后，Hyperspace服务器就会向BDB集群中的Master服务器发起事务操作。BDB服务器在接收到该事务请求后，会在集群内部发起一轮事务请求投票流程，一旦BDB集群内部过半的服务器成功应用了该事务操作，就会反馈Hyperspace服务器更新已经成功，再由Hyperspace响应上层的Master模块。

#### Active Hyperspace选举

Active选举过程的核心逻辑就是根据所有服务器上事务日志的更新时间来确定哪个服务器的数据最新——事务日志更新时间越新，那么这台服务器被选举为Active Hyperspace的可能性就越大。

## 小结

Paxos算法超强的容错能力和分布式数据一致性的可靠保证，使其在工业界得到了广泛的应用。

# 第4章 Zookeeper与Paxos

Apache Zookeeper是由Apache Hadoop的子项目发展而来，于2010年11月正式成为了Apache的顶级项目。Zookeeper为分布式应用提供了高效且可靠的分布式协调服务，提供了诸如统一命名服务、配置管理和分布式锁等分布式的基础服务。在解决分布式数据一致性方面，ZooKeeper并没有直接采用Paxos算法，而是采用了一种被称为ZAB(ZooKeeper Atomic Broadcast)的一致性协议。

## 4.1 初识ZooKeeper

### 4.1.1 ZooKeeper介绍

ZooKeeper是一个开放源代码的分布式协调服务，由知名互联网公司雅虎创建，是Google Chubby的开源实现。ZooKeeper的设计目标是将那些复杂且容易出错的分布式一致性服务封装起来，构成一个高效可靠的原语集，并以一系列简单易用的接口提供给用户使用。

#### ZooKeeper是什么

ZooKeeper是一个典型的分布式数据一致性的解决方案，分布式应用程序可以基于它实现诸如数据发布/订阅、负载均衡、命名服务、分布式协调/通知、集群管理、Master选举、分布式锁和分布式队列等功能。ZooKeeper可以保证如下分布式一致性特性：

*   顺序一致性
*   原子性
*   单一视图（Single System Image）
*   可靠性
*   实时性（在一段时间内，最终一致）

#### ZooKeeper的设计目标

ZooKeeper致力于提供一个高性能、高可用，且具有严格的顺序访问控制能力（主要是写操作的严格顺序性）的分布式协调服务。

##### 目标一：简单的数据模型

ZooKeeper使得分布式程序能够通过一个共享的、树型结构的名字空间来进行相互协调。

##### 目标二：可以构建集群

![](/img/notes/distribute/paxosZookeeper/zookeeper_cluster.png)

ZooKeeper的客户端程序会选择和集群中任意一台机器共同创建一个TCP连接，而一旦客户端和某台ZooKeeper服务器之间的连接断开后，客户端会自动连接到集群中的其他机器。

##### 目标三：顺序访问

对于来自客户端的每个更新请求，ZooKeeper都会分配一个全局唯一的递增编号，这个编号反映了所有事务操作的先后顺利，应用程序可以使用ZooKeeper的这个特性来实现更高层次的同步原语。

##### 目标四：高性能

由于ZooKeeper将全量数据存储在内存中，并直接服务于客户端的所有非事务请求，因此它尤其适用于以读操作为主的应用场景。

3台3.4.3版本的ZooKeeper服务器组成集群进行性能压测，100%读请求的场景下压测结果是12-13W的QPS。

### 4.1.2 ZooKeeper从何而来

ZooKeeper最早起源于雅虎研究院的一个研究小组。

### 4.1.3 ZooKeeper的基本概念

#### 集群角色

在ZooKeeper中，引入了Leader、Follower和Observer三种角色。ZooKeeper集群中的所有机器通过一个Leader选举过程来选定一台被称为"Leader"的机器，Leader服务器为客户端提供读和写服务。除Leader外，其他机器包括Follower和Observer。Follower和Observer都能够提供读服务。唯一的区别在于，Observer机器不参与Leader选举过程，也不参与写操作的“过半写成功”策略，因此Observer可以在不影响写性能的情况下提升集群的读性能。

#### 会话（Session）

在ZooKeeper中，一个客户端连接是指客户端和服务器之间的一个TCP长连接。客户端启动的时候，首先会与服务器建立一个TCP连接，从第一次连接建立开始，客户端会话的生命周期也开始了，通过这个连接，客户端能够通过心跳检测与服务器保持有效的会话，也能够向ZooKeeper服务器发送请求并接受响应，同时还能够通过该连接接收来自服务器的Watch事件通知。

Session的sessionTimeout值用来设置一个客户端会话的超时时间。由于服务器压力太大、网络故障或是客户端主动断开连接等各种原因导致客户端连接断开时，只要在sessionTimeout规定的时间内能够重新连接上集群中任意一台服务器，那么之前创建的会话依然有效。

#### 数据节点（Znode）

ZooKeeper中的两类节点：

*   机器节点：构成集群的机器
*   数据节点（Znode）：数据模型中的数据单元。

ZooKeeper将所有数据存储在内存中，数据模型是一棵树（ZNode Tree），由斜杠（/）进行分割的路径，就是一个Znode，例如/foo/path1。每个ZNode上都会保存自己的数据内容，同时还会保存一些列属性信息。

在ZooKeeper中，ZNode可以分为持久节点和临时节点两类。

ZooKeeper还允许用户为每个节点添加一个特殊的属性：SEQUENTIAL。一旦节点被标记上这个属性，那么在这个节点被创建的时候，ZooKeeper会自动在其节点后面追加上一个整型数字，这个整型数字是一个由父节点维护的自增数字。

#### 版本

ZooKeeper的每个ZNode上都会存储数据，对应于每个ZNode，ZooKeeper都会为其维护一个叫作Stat的数据结构，Stat中记录了这个ZNode的三个数据版本，分别是version（当前ZNode版本）、cversion（当前ZNode子节点的版本）和aversion（当前ZNode的ACL版本）。

#### Watcher

ZooKeeper允许用户在指定节点上注册一些Watcher（事件监听器），并且在一些特定事件触发的时候，ZooKeeper服务器会将事件通知到感兴趣的客户端上去，该机制是ZooKeeper实现分布式协调服务的重要特性。

#### ACL

ZooKeeper采用ACL（Access Control Lists）策略来进行权限控制，类似于UNIX文件系统的权限控制。ZooKeeper定义的如下5种权限：

*   CREATE：创建子节点的权限。
*   READ：获取节点数据和子节点列表的权限。
*   WRITE：更新节点数据的权限。
*   DELETE：删除子节点的权限。
*   ADMIN：设置节点ACL的权限。

注：CREATE和DELETE这两种权限都是针对子节点的权限控制。


### 4.1.4 为什么选择ZooKeeper

*   除了ZooKeeper之外，目前还没有一个成熟稳定且被大规模应用的解决方案。ZooKeeper无论从性能、易用性还是稳定性上来说，都已经达到了一个工业级产品的标准。
*   ZooKeeper是开放源代码的。
*   ZooKeeper是免费的
*   ZooKeeper已经得到了广泛的应用。诸如Hadoop、HBase、Storm和Solr等越来越多的大型分布式项目都已经将ZooKeeper作为其核心组件，用于分布式协调。

## 4.2 ZooKeeper的ZAB协议

### 4.2.1 ZAB协议

ZooKeeper使用了一种称为ZooKeeper Atomic Broadcast（ZAB，ZooKeeper原子消息广播协议）的协议作为其数据一致性的核心算法。

ZAB协议是为分布式协调服务ZooKeeper专门设计的一种支持崩溃恢复的原子广播协议。ZAB协议并不像Paxos算法那样，是一种通用的分布式一致性算法，它是一种特别为ZooKeeper设计的崩溃可恢复的原子消息广播算法。

基于ZAB协议，ZooKeeper实现了一种主备模式的系统架构来保持集群中各副本之间的数据的一致性。

*   ZAB协议的这个主备模型架构保证了同一时刻集群中只能够有一个主进程来广播服务器的状态变更。
*   ZAB协议必须能够保证一个全局的变更序列被顺序应用。
*   ZAB协议还需要做到在当前主进程出现上述异常情况的时候，依旧能够正常工作。

ZAB协议的核心是定义了对于那些会改变ZooKeeper服务器数据状态的事务请求的处理方式，即：

*   *所有事务请求必须由一个全局唯一的服务器来协调处理，这样的服务器被称为Leader服务器，而余下的其他服务器则称为Follower服务器。Leader服务器负责将一个客户端事务请求转换成一个事务Proposal（提议），并将该Proposal分发给集群中所有的Follower服务器。之后Leader服务器需要等待所有Follower服务器的反馈，一旦超过半数的Follower服务器进行了正确的反馈后，那么Leader就会再次向所有的Follower服务器分发Commit消息，要求其将前一个Proposal进行提交。*

### 4.2.2 协议介绍

ZAB协议包括两种基本模式：崩溃恢复和消息广播。

#### 消息广播

![](/img/notes/distribute/paxosZookeeper/zab_broadcast.jpg)

#### 崩溃恢复

ZAB协议需要一个高效且可靠的Leader选举算法，从而确保能够快速地选举出新的Leader。同时，Leader选举算法不仅仅需要让Leader自己知道其自身已经被选举为leader，同时还需要让集群中的所有其他机器也能够快速地感知到选举产生的新的Leader服务器。

##### 基本特性

*   ZAB协议需要确保那些已经在Leader服务器上提交的事务最终被所有服务器都提交。
*   ZAB协议需要确保丢弃那些只在Leader服务器上被提出的事务。

如果让Leader选举算法能够保证新选举出来的Leader服务器拥有集群中所有机器最高编号（即ZXID最大）的事务Proposal，那么就可以保证这个新选举出来的Leader一定具有所有已经提交的提案。更为重要的是，如果让具有最高编号事务Proposal的机器来称为Leader，就可以省去Leader服务器检查Proposal的提交和丢弃工作的这一步操作了。

##### 数据同步

在ZAB协议的事务编号ZXID设计中，ZXID是一个64位的数字，其中低32位可以看作是一个简单的单调递增的计数器，针对客户端的每一个事务请求，Leader服务器在产生一个新的事务Proposal的时候，都会对该计数器进行加1操作；而高32位则代表了Leader周期epoch的编号，每当选举产生一个新的Leader服务器，就会从这个Leader服务器上取出其本地日志中最大事务Proposal的ZXID，并从该ZXID中解析出对应的epoch值，然后再对其进行加1操作，之后就会以此编号作为新的epoch，并将低32位置0来开始生成新的ZXID。

### 4.2.3 深入ZAB协议

#### 算法描述

整个ZAB协议主要包括消息广播和崩溃恢复两个过程，进一步可以细分为三个阶段，分别是发现（Discovery）、同步（Synchronization）和广播（Broadcast）阶段。组成ZAB协议的每一个分布式进程，会循环地执行这三个阶段，我们将这样一个循环称为一个主进程周期。

术语表：

*   F<sub>p</sub>：Follower f处理过的最后一个事务Proposal
*   F<sub>zxid</sub>：Follower f处理过的历史事务Proposal中最后一个事务Proposal的事务标识ZXID
*   h<sub>f</sub>：每一个Follower f通常都已经处理（接受）了不少事务Proposal，并且会有一个针对已经处理过的事务的集合，将其表示为h<sub>f</sub>，表示Follower f已经处理过的事务序列
*   I<sub>e</sub>：初始化历史记录，在某一个主进程周期epoch e中，当准Leader完成阶段一之后，此时它的h<sub>f</sub>就被标记为I<sub>e</sub>。

##### 阶段一：发现

阶段一主要就是Leader选举过程，用于在多个分布式进程中选举出主进程，准Leader L和Follower F的工作流程分别如下：

*   步骤F.1.1 Follower F将自己最后接受的事务Proposal的epoch值CEPOCH(F<sub>p</sub>)发送给准Leader L。
*   步骤L.1.1 当接收到来自过半Follower的CEPOCH(F<sub>p</sub>)消息后，准Leader L会生成NEWEPOCH(e')消息给这些过半的Follower。关于这个epoch值e'，准Leader L会从所有接收到的CEPOCH(F<sub>p</sub>)消息中选取出最大的epoch值，然后对其进行加1操作，即为e'。
*   步骤F.1.2 当Follower接收到来自准Leader L的NEWEPOCH(e')消息后，如果其检测到当前的CEPOCH(F<sub>p</sub>)值小于e'，那么就会将CEPOCH(F<sub>p</sub>)赋值为e'，同时向这个准Leader L反馈Ack消息。在这个反馈消息（ACK-E(F<sub>p</sub>, h<sub>f</sub>))中，包含了当前该Follower的epoch CEPOCH(F<sub>p</sub>)，以及该Follower的历史事务Proposal集合：h<sub>f</sub>。

当Leader L接收到来自过半Follower的确认消息Ack之后，Leader L就会从这过半服务器中选取出一个Follower F，并使用其作为初始化事务集合I<sub>e'</sub>。

##### 阶段二：同步

在完成发现流程之后，就进入了同步阶段。在这一阶段中，Leader L和Follower F的工作流程分别如下：

*   步骤L.2.1 Leader L会将e'和Ie'以NEWLEADER(e', I<sub>e'</sub>)消息的形式发送给所有Quorum中的Follower。
*   步骤F.2.1 当Follower接收到来自Leader L的NEWLEADER(e', I<sub>e'</sub>)消息后，如果Follower发现CEPOCH(F<sub>p</sub>) &lt;&gt; e'，那么直接进入下一轮循环，因为此时Follower发现自己还在上一轮，或者更上轮，无法参与本轮的同步。如果CEPOCH(F<sub>p</sub>) = e'，那么Follower就会执行事务应用操作。最后，Follower会反馈给Leader，表明自己已经接受并处理了所有I<sub>e'</sub>中的事务Proposal。
*   步骤L.2.2 当Leader接收到来自过半Follower针对NEWLEADER(e', I<sub>e'</sub>)的反馈消息后，就会向所有的Follower发送Commit消息。至此Leader完成阶段二。
*   步骤F.2.2 当Follower收到来自Leader的Commit消息后，就会依次处理并提交所有在I<sub>e''</sub>中未处理的事务。至此Follower完成阶段二。

##### 阶段三：广播

完成同步阶段之后，ZAB协议就可以正式开始接收客户端新的事务请求，并进行消息广播流程。

*   步骤L.3.1 Leader L接收到客户端新的事务请求后，会生成对应的事务Proposal，并根据ZXID的顺序向所有Follower发送提案&lt;e', &lt;v, z&gt;&gt;，其中epoch(z) = e'。
*   步骤F.3.1 Follower根据消息接收的先后次序来处理这些来自Leader的事务Proposal，并将他们追加到h<sub>f</sub>中去，之后再反馈给Leader。
*   步骤L.3.2 当Leader接收到来自过半Follower针对事务Proposal&lt;e', &lt;v, z&gt;&gt;的Ack消息后，就会发送Commit&lt;e', &lt;v, z&gt;&gt;消息给所有的Follower，要求它们进行事务的提交。
*   步骤F.3.2 当Follower F接收到来自Leader的Commit&lt;e', &lt;v, z&gt;&gt;消息后，就会开始提交事务Proposal&lt;e', &lt;v, z&gt;&gt;。

#### 运行分析

在ZAB协议的设计中，每一个进程都有可能处于以下三种状态之一：

*   LOOKING：Leader选举阶段
*   FOLLOWING：Follower服务器和Leader保持同步状态
*   LEADING：Leader服务器作为主进程领导状态

我们将一个可用的Leader定义如下：

*   *如果一个准Leader L<sub>e</sub>接收到来自过半的Follower进程针对L<sub>e</sub>的NEWLEADER(e, L<sub>e</sub>)反馈消息，那么L<sub>e</sub>就成为了周期e的Leader。*

### 4.2.4 ZAB与Paxos算法的联系和区别

两者的联系：

*   两者都存在一个类似于Leader进程的角色，由其负责协调多个Follower进程的运行。
*   Leader进程都会等待超过半数的Follower做出正确的反馈后，才会将一个提案进行提交。
*   在ZAB协议中，每个Proposal中都包含了一个epoch值，用来代表当前的Leader周期，在Paxos算法中，同样存在这样的一个标识，只是名字变成了Ballot。

ZAB协议和Paxos算法的本质区别在于，两者的设计目标不太一样。ZAB协议主要用于构建一个高可用的分布式数据主备系统，例如ZooKeeper，而Paxos算法则是用于构建一个分布式的一致性状态机系统。

## 小结

# 第5章 使用ZooKeeper

## 5.1 部署与运行

本章的讲解针对的ZooKeeper的官方版本是3.4.3。

### 5.1.1 系统环境

**操作系统**：不建议在FreeBSD系统上部署生产环境的ZooKeeper服务器。

**Java环境**：1.6或以上版本的Java

### 5.1.2 集群与单机

#### 集群模式

1.  准备Java运行环境
2.  下载ZooKeeper安装包
3.  配置文件zoo.cfg
4.  创建myid文件
5.  按照相同的步骤，为其他机器都配置上zoo.cfg和myid文件
6.  启动服务器

    **$ sh zkServer.sh start**

7.  验证服务器

    **$   telnet 127.0.0.1 2181**
    
    Trying 127.0.0.1...
    
    Connected to localhost.localdomain (127.0.0.1).
    
    Escape character is '^]'.
    
    **stat**
    
    ......

#### 单机模式

和集群模式的唯一区别就在机器列表上，在单机模式的zoo.cfg文件中，只有server.1这一项。

集群模式和单机模式下输出的服务器验证信息基本一致，只有Mode属性不一样。在集群模式中，Mode显示的是leader，其实还有可能是follower。而在单机模式中，Mode显示的是standalone。

#### 伪集群模式

集群所有的机器都在一台机器上，但是还是以集群的特性来对外提供服务。

### 5.1.3 运行服务

#### 启动服务

*   Java命令行
*   使用ZooKeeper自带的启动脚本来启动ZooKeeper

ZooKeeper可执行脚本：

*   zkCleanup：清理ZooKeeper历史数据，包括事务日志文件和快照数据文件
*   zkCli：ZooKeeper的一个简易客户端
*   zkEnv：设置ZooKeeper的环境变量
*   zkServer：ZooKeeper服务器的启动、停止和重启脚本

#### 停止服务

    $ sh zkServer.sh stop

#### 常见异常

*   端口被占用：**Address already in use**
*   磁盘没有剩余空间：**No space left on device**
*   无法找到myid文件：**Invalid config, exiting abnormally**
*   集群中其他机器Leader选举端口未开：快速启动集群中的其他机器即可。

## 5.2 客户端脚本

    $ sh zkCli.sh -server ip:port

### 5.2.1 创建

使用create命令，可以创建一个ZooKeeper节点。

    create [-s] [-e] path data acl

其中，-s或-e分别指定节点特性：顺序或临时节点。默认情况下，即不添加-s或-e参数的，创建的是持久节点。create命令的最后一个参数是acl，用来进行权限控制的，缺省情况下，不做任何权限控制。

### 5.2.2 读取

#### ls

列出ZooKeeper指定节点下的所有子节点（第一级）。

    ls path [watch]

第一次部署的ZooKeeper集群，默认在根节点"/"下面有一个叫作/zookeeper的保留节点。

#### get

使用get命令，可以获取ZooKeeper指定节点的数据内容和属性信息。

    get path [watch]

结果的第一行是节点/zk-book的数据内容，其他几行则是创建该节点的事务ID（cZxid）、最后一次更新该节点的事务ID（mZxid）和最后一次更新该节点的时间（mtime）等属性信息。

### 5.2.3 更新

使用set命令，可以更新指定节点的数据内容。
    
    set path data [version]

### 5.2.4 删除

使用delete命令，可以删除ZooKeeper上的指定节点。

    delete path [version]

注：要想删除一个指定节点，该节点必须没有子节点存在。

## 5.3 Java客户端API使用

### 5.3.1 创建会话

客户端可以通过创建一个ZooKeeper(org.apache.zookeeper.ZooKeeper)实例来连接ZooKeeper服务器。

ZooKeeper构造方法参数说明：

*   connectString：ZooKeeper服务器列表，由英文状态逗号分开的host:port字符串组成，每一个都代表一台ZooKeeper机器。
*   sessionTimeout：会话的超时时间，是一个以“毫秒”为单位的整型值。
*   watcher：ZooKeeper允许客户端在构建方法中传入一个接口Watcher(org.apache.zookeeper.Watcher)的实现类对象作为默认的Watcher事件通知处理器。
*   canBeReadOnly：用于标识当前会话是否支持"read-only（只读）"模式。
*   sessionId和sessionPasswd：分别代表会话ID和会话秘钥。这两个参数能够唯一确定一个会话，同时客户端使用这两个参数可以实现客户端会话复用，从而达到恢复会话的效果。具体使用方法是，第一次连接上ZooKeeper服务器时，通过调用ZooKeeper对象实例的以下两个接口，即可获得当前会话的ID和秘钥：
    *   long getSessionId();
    *   byte[] getSessionPasswd();

### 5.3.2 创建节点

有如下两个接口（第一个同步，第二个异步）：

    String create(final String path, byte data[], List<ACL> acl, CreateMode createMode);
    void create(final String path, byte data[], List<ACL> acl, CreateMode createMode, StringCallback cb, Object ctx);

参数说明如下：

*   path：需要创建的数据的节点路径
*   data[]：一个字节数组，是节点创建后的初始内容
*   acl：节点的ACL策略
*   createMode：节点类型，是一个枚举类型，通常有4种可选的节点类型：
    *   持久（PERSISTENT）
    *   持久顺序（PERSISTENT_SEQUENTIAL）
    *   临时（EPHEMERAL）
    *   临时顺序（EPHEMERAL_SEQUENTIAL）
*   cb：注册一个异步回调函数，开发人员需要实现StringCallback接口，当服务器节点创建完毕后，ZooKeeper客户端就会自动调用如下方法：

    <code>void processResult(int rc, String path, Object ctx, String name);</code>

*   ctx：用于传递一个对象，可以在回调方法执行的时候使用，通常是放一个上下文（Context）信息

ZooKeeper不支持递归创建，即无法在父节点不存在的情况下创建一个子节点。

如果应用场景没有太高的权限要求，那么可以不关注这个参数，只需要在acl参数中传入参数Ids.OPEN_ACL_UNSAFE，这就表明之后对这个节点的任何操作都不受权限控制。

回调方法processResult方法参数说明：

*   rc：Result Code，服务端响应码。
    *   0（OK）：接口调用成功。
    *   -4（CnnectionLoss）：客户端和服务端连接已断开。
    *   -110（NodeExists）：指定节点已存在。
    *   -112（SessionExpired）：会话已过期
*   path：接口调用时传入API的数据节点的节点路径参数值
*   ctx：接口调用时传入API的ctx参数值
*   name：实际在服务端创建的节点名。

### 5.3.3 删除节点

有如下同步和异步两个接口：

    public void delete(final String path, int version)
    public void delete(final String path, int version, VoidCallback cb, Object ctx)

参数说明：

*   path：指定数据节点的节点路径
*   version：指定节点的数据版本
*   cb：注册一个异步回调函数
*   ctx：用于传递上下文信息的对象

在ZooKeeper中，只允许删除叶子节点。

### 5.3.4 读取数据

#### getChildren

获取一个节点的所有子节点

参数说明：

*   path：指定数据节点的节点路径
*   watcher：注册的Watcher。一旦在本次子节点获取之后，子节点列表发生变更的话，那么就会向客户端发送通知。该参数允许传入null
*   watch：表名是否需要注册一个Watcher。如果这个参数是true，那么ZooKeeper客户端会自动使用默认Watcher；如果是false，表明不需要注册Watcher
*   cb：注册一个异步回调函数
*   ctx：用于传递上下文信息的对象
*   stat：指定数据节点的节点状态信息。用法是在接口中传入一个旧的stat变量，该stat变量会在方法执行过程中，被来自服务端响应的新stat对象替换

当注册了Watcher，当有子节点被添加或是删除时，服务端就会向客户端发送一个NodeChildrenChanged(EventType.NodeChildrenChanged)类型的事件通知。需要注意的是，在服务端发送给客户端的事件通知中，是不包含最新的节点列表的，客户端必须主动重新进行获取。

由于Watcher通知是一次性的，即一旦触发一次通知后，该Watcher就失效了，因此客户端需要反复注册Watcher。

#### getData

获取一个节点的数据内容。

参数说明：

*   path：指定数据节点的节点路径
*   watcher：注册的Watcher。
*   stat：指定数据节点的节点状态信息。
*   watch：表明是否需要注册一个Watcher。
*   cb：注册一个异步回调函数
*   ctx：用于传递上下文信息的对象

数据内容或是数据版本变化，都会触发服务端的NodeDataChanged通知。

### 5.3.5 更新数据

更新一个节点的数据内容。

两个接口（同步、异步）：

    Stat setData(final String path, byte data[], int version);
    void setData(final String path, byte data[], int version, StatCallback cb, Object ctx);

version参数的意义：CAS

### 5.3.6 检测节点是否存在

4个接口：

    public Stat exists(final String path, Watcher watcher);
    public Stat exists(String path, boolean watch);
    public void exists(final String path, Watcher watcher, StatCallback cb, Object ctx);
    public void exists(String path, boolean watch, StatCallback cb, Object ctx);

如果在调用接口时注册Watcher的话，还可以对节点是否存在进行监听——一旦节点被创建、被删除或是数据被更新，都会通知客户端。

*   无论指定节点是否存在，通过调用exists接口都可以注册Watcher。
*   exists接口中注册的Watcher，能够对节点创建、节点删除和节点数据更新事件进行监听。
*   对于指定节点的子节点的各种变化，都不会通知客户端。

### 5.3.7 权限控制

开发人员如果要使用ZooKeeper的权限控制功能，需要在完成ZooKeeper会话创建后，给该会话添加上相关的权限信息（AuthInfo）。ZooKeeper客户端提供了响应的API接口来进行权限信息的设置，如下：

    addAuthInfo(String scheme, byte[] auth);

参数说明：

*   scheme：权限控制模式，分为world、auth、digest、ip和super
*   auth：具体的权限信息

当客户端对一个数据节点添加了权限信息后，对于删除操作而言，其作用范围是其子节点。

## 5.4 开源客户端

### 5.4.1 ZkClient

ZkClient是Github上一个开源的ZooKeeper客户端。

ZkClient在ZooKeeper原生API接口之上进行了包装，是一个更易用的ZooKeeper客户端。同时，ZkClient在内部实现了诸如Session超时重连，Watcher反复注册等功能，使得ZooKeeper客户端的这些繁琐的细节工作对开发人员透明。

#### 创建会话

参数说明：

*   zkServers：指ZooKeeper服务器列表
*   sessionTimeout：会话超时时间，单位为毫秒
*   connectionTimeout：连接创建超时时间，单位为毫秒
*   connection：IZkConnection接口的实现类
*   zkSerializer：自定义序列化器

IZkConnection接口是对ZooKeeper原生接口最直接的包装，也是和ZooKeeper最直接的交互层，里面包含了添、删、改、查等一系列接口的定义。

ZkClient中定义了ZkSerializer接口，允许用户传入一个序列化实现。

ZkClient构造方法中，不再提供传入Watcher对象的参数了。ZkClient引入了大多数Java程序都使用过的Listener来实现Watcher注册。

#### 创建节点

createEphemeral接口是创建临时节点，而createPersistentSequential接口则是创建持久顺序节点。

通过createParents这个参数，ZkClient能够在内部帮助我们递归建立父节点。

#### 删除节点

deleteRecursive这个接口将自动帮我们完成逐层删除节点的工作。

#### 读取数据

##### getChildren

    List<String> getChildren(String path);

可以通过如下API来进行注册监听：

    List<String> subscribeChildChanges(String path, IZkChildListener listener);

Listener接口的定义：

    public interface IZkChildListener {
        public void handleChildChange(String parentPath, List<String> currentChildrens);
    }

结论：

*   客户端可以对一个不存在的节点进行子节点变更的监听。
*   一旦客户端对一个节点注册了子节点列表变更监听之后，那么当该节点的子节点列表发生变更的时候，服务端都会通知客户端，并将最新的子节点列表发送给客户端。
*   该节点本身的创建或删除也会通知到客户端。

ZkClient的Listener不是一次性的，客户端只需要注册一次就会一直生效。

##### getData

接口：

    <T extends Object> T readData(String path);
    <T extends Object> T readData(String path, boolean returnNullIfPathNotExists);
    <T extends Object> T readData(String path, Stat stat);

该接口对服务端时间的监听，同样是通过注册指定的Listener来实现的：

    public interface IZkDataListener {
        public void handleDataChange(String dataPath, Object data) throws Exception;
        public void handleDataDeleted(String dataPath) throws Exception;
    }
    
#### 更新数据

接口：
    
    void writeData(String path, Object data);
    void writeData(final String path, Object data, final int expectedVersion);

#### 检测节点是否存在

接口：

    boolean exists(final String path);

### 5.4.2 Curator

* Guava is to Java what Curator is to ZooKeeper *

除了封装一些开发人员不需要特别关注的底层细节之外，Curator还在ZooKeeper原生API的基础上进行了包装，提供了一套易用性和可读性更强的Fluent风格的客户端API框架。

#### 创建会话

RetryPolicy

Fluent风格：

    CuratorFrameworkFactory.builder()
                        .conncectString("xxx:2181")
                        .sessionTimeoutMs(5000)
                        .retryPolicy(retryPolicy)
                        .namespace("base")
                        .build();
    client.start();

以上代码片段中定义了某一个客户端的独立命名空间为base。

#### 创建节点

创建一个节点，初始内容为空：

    client.create().forPath(path);

创建一个节点，附带初始内容：

    client.create().forPath(path, "init".getBytes());

创建一个临时节点，初始内容为空：

    client.create().withMode(CreateMode.EPHEMERAL).forPath(path);

创建一个临时节点，并自动递归创建父节点：

    client.create().creatingParentsIfNeeded().withMode(CreateMode.EPHEMERAL).forPath(path);

注：由于在ZooKeeper中规定了所有非叶子节点必须为持久节点，调用上面这个API之后，只有path参数对应的数据节点是临时节点，其父节点均为持久节点。

#### 删除节点

删除一个几点：

    client.delete().forPath(path);

删除一个节点，并递归删除其所有子节点：

    client.delete().deletingChildrenIfNeeded().forPath(path);

删除一个节点，强制保证删除：

    client.delete().guaranteed().forPath(path);

#### 读取数据

读取一个节点的数据内容：

    client.getData().forPath(path);

读取一个节点的数据内容，同时获取到该节点的stat：

    client.getData().storingStatIn(stat).forPath(path);

#### 更新数据

更新一个节点的数据内容：

    client.setData().forPath(path);

更新一个节点的数据内容，强制指定版本更新：

    client.setData().withVersion(version).forPath(path);

#### 异步接口

Curator中引入了BackgroundCallback接口，用来处理异步接口调用之后服务端返回的结果信息，其接口定义如下：

    public interface BackgroundCallback {
    
        public void processResult(CuratorFramework client, CuratorEvent event) throws Exception;
    }

参数说明：

*   client：当前客户端实例
*   event：服务端事件，包括事件类型（CuratorEventType）和响应码（int）

#### 典型使用场景

curator-recipes

##### 事件监听

*   NodeCache：监听指定ZooKeeper数据节点本身的变化
*   PathChildrenCache：监听指定ZooKeeper数据节点的子节点变化

##### Master选举

思路：选择一个根节点，多台机器同时向该节点创建一个子节点，利用ZooKeeper特性，最终只有一台机器能够创建成功，成功的那台机器就作为Master。

LeaderSelector

##### 分布式锁

核心接口：

    public interface InterProcessLock {

        public void acquire() throws Exception;
        public void release() throws Exception;
    }

##### 分布式计数器

实现思路：指定一个ZooKeeper数据节点作为计数器，多个应用实例在分布式锁的控制下，通过更新该数据节点的内容来实现计数功能。

DistributeAtomicInteger类

##### 分布式Barrier

Barrier是一种用来控制多线程之间同步的经典方式，在JDK中也自带了CyclicBarrier实现。

Curator中提供的DistributedBarrier

#### 工具

##### ZKPaths

ZKPaths提供了一些简单的API来构建ZNode路径、递归创建和删除节点等。

##### EnsurePath

EnsurePath提供了一种能够确保数据节点存在的机制，多用于这样的业务场景中：*上层业务希望对一个数据节点进行一些操作，但是操作之前需要确保该节点存在。*

EnsurePath采取了静默的节点创建方式，其内部实现就是试图创建指定节点，如果节点已经存在，那么就不进行任何操作，也不对外抛出异常，否则正常创建数据节点。

##### TestingServer

TestingServer允许开发人员非常方便地启动一个标准的ZooKeeper服务器，并以此来进行一系列的单元测试。

##### TestingCluster

TestingCluster是一个可以模拟ZooKeeper集群环境的Curator工具类，能够便于开发人员在本地模拟由n台机器组成的集群环境。

## 小结

# 第6章 ZooKeeper的典型应用场景

## 6.1 典型应用场景及实现

### 6.1.1 数据发布/订阅

ZooKeeper采用的是推拉相结合的方式：客户端向服务端注册自己需要关注的节点，一旦该节点的数据发生变更，那么服务端就会向相应的客户端发送Watcher事件通知，客户端接收到这个消息通知后，需要主动到服务端获取最新的数据。

### 6.1.2 负载均衡

动态、自动化的DNS服务

### 6.1.3 命名服务

Name Service

UUID（Universally Unique Identifier）：通用唯一识别码

GUID(Global Unique Identifier)：全局唯一标识符，是UUID的一种典型实现。

UUID的缺陷：

*   长度过长
*   含义不明

![](/img/notes/distribute/paxosZookeeper/guid_zookeeper.png)

使用ZooKeeper生成唯一ID的基本步骤：

1.  所有客户端都会根据自己的任务类型，在指定类型的任务下面通过调用create()接口来创建一个顺序节点。
2.  节点创建完毕后，create()接口会返回一个完整的节点名。
3.  客户端拿到这个返回值后，拼接上type类型，，就可以作为一个全局唯一的ID了。

在ZooKeeper中，每一个数据节点都能够维护一份子节点的顺序，当客户端对其创建一个顺序子节点的时候ZooKeeper会自动以后缀的形式在其子节点上添加一个序号，在这个场景中就是利用了ZooKeeper的这个特性。

### 6.1.4 分布式协调/通知

#### MySQL数据复制总线：Mysql_Replicator

热备份策略：“小序号优先”策略

#### 一种通用的分布式系统机器间通信方式

*   心跳检测：基于ZooKeeper的临时节点特性，可以让不同的机器都在ZooKeeper的一个指定节点下创建临时子节点，不同的机器之间可以根据这个临时节点来判断对应的客户端机器是否存活。
*   工作进度汇报：在ZooKeeper上选择一个节点，每个任务客户端都在这个节点下面创建临时子节点。
*   系统调度

总之，使用ZooKeeper来实现分布式系统机器间的通信，不仅能省去大量底层网络通信和协议设计上重复的工作，更为重要的一点是大大降低了系统之间的耦合，能够非常方便地实现异构系统之间的灵活通信。

### 6.1.5 集群管理

*   希望知道当前集群中究竟有多少机器在工作
*   对集群中每台机器的运行时状态进行数据收集
*   对集群中机器进行上下线操作

在传统的基于Agent的分布式集群管理体系中，都是通过在集群中的每台机器上部署一个Agent，由这个Agent负责主动向指定的一个监控中心系统汇报自己所在机器的状态。缺陷：

*   大规模升级困难
*   统一的Agent无法满足多样的需求
*   编程语言多样性

ZooKeeper具有以下两大特性：

*   客户端如果对ZooKeeper的一个数据节点注册Watcher监听，那么当该数据节点的内容或是其子节点列表发生变更时，ZooKeeper服务器就会向订阅的客户端发送变更通知。
*   对在ZooKeeper上创建的临时节点，一旦客户端与服务器之间的会话失效，那么该临时节点也就被自动清除。

#### 分布式日志收集系统

两大问题：

*   变化的日志源机器
*   变化的收集器机器

![](/img/notes/distribute/paxosZookeeper/distribute_log_collector_zookeeper.png)

#### 在线云主机管理

需求点：

*   如何快速地统计当前生产环境一共有多少台机器？
*   如何快速地获取到机器上/下线的情况？
*   如何实时监控集群中每台主机的运行时状态？

借助ZooKeeper来实现的方式，不仅能够实时地检测到集群中机器的上/下线情况，而且能够实时地获取到主机的运行时信息，从而能够构建出一个大规模集群的主机图谱。

### 6.1.6 Master选举

第一个成功创建指定临时节点的客户端所在的机器就成为了Master

### 6.1.7 分布式锁

#### 排他锁

Exclusive Locks，简称X锁

##### 定义锁

在通常的Java开发编程中，有两种常见的方式可以用来定义锁，分别是synchornized机制和JDK5提供的ReentantLock。

在ZooKeeper中，是通过ZooKeeper上的数据节点来表示一个锁。

##### 获取锁

所有的客户端都会视图通过调用create()接口，创建临时子节点，ZooKeeper会保证在所有的客户端中，最终只有一个客户端能够创建成功，那么就可以认为该客户端获得了锁。同时，所有没有获取到锁的客户端就需要到节点上注册一个节点变更的Watcher监听，以便实时监听到lock节点的变更情况。

##### 释放锁

两种情况：

*   当前获取锁的客户端机器发生宕机，那么ZooKeeper上的这个临时节点就会被移除
*   正常执行完业务逻辑后，客户端就会主动将自己创建的临时节点删除。

#### 共享锁

Shared Locks，简称S锁

共享锁和排他锁最根本的区别在于，加上排他锁后，数据对象只对一个事务可见，而加上共享锁后，数据对所有事务都可见。

##### 定义锁

通过ZooKeeper上的数据节点来表示一个锁。

##### 获取锁

在需要获取共享锁时，所有客户端都会到锁节点下面创建一个临时顺序节点，如果当前是读请求，那么就创建读的节点；如果是写请求，那么就创建写的节点。

##### 判断读写顺序

1.  创建完节点后，获取父节点下的所有子节点，并对该节点注册子节点变更的Watcher监听。
2.  确定自己的节点序号在所有子节点中的顺序。
3.  对于读请求：
    *   如果没有比自己序号小的子节点，或是所有比自己序号小的子节点都是读请求，那么表名自己已经成功获取到了共享锁，同时开始执行读取逻辑。
    *   如果比自己序号小的子节点中有写请求，那么就需要进入等待。
4.  对于写请求：
    *   如果自己不是序号最小的子节点，那么就需要进入等待。
5.  接收到Watcher通知后，重复步骤1。

##### 释放锁

释放锁的逻辑和排他锁一致。

##### 羊群效应

在这整个分布式锁的竞争过程中，大量的"Watcher 通知"和"子节点列表获取"两个操作重复执行，客户端无端地接收到过多和自己并不相关的事件通知。

核心逻辑：判断自己是否是所有子节点中序号最小的

改进思路：每个节点对应的客户端只需要关注比自己序号小的那个相关节点的变更情况就可以了。

##### 改进后的分布式锁实现

1.  客户端调用create()方法创建临时顺序子节点。
2.  客户端调用getChildren()接口来获取所有已经创建的子节点列表（不注册任何Watcher）。
3.  如果无法获取共享锁，那么就调用exist()来对比逼自己小的那么个节点注册Watcher，具体的：
    *   读请求：向比自己序号小的最后一个写请求节点注册Watcher监听。
    *   写请求：向比自己序号小的最后一个节点注册Watcher监听。
4.  等待Watcher通知，继续进入步骤2。

##### 注意

在具体的实际开发过程中，我们提倡根据具体的业务场景和集群规模来选择适合自己的分布式锁实现。

### 6.1.8 分布式队列

两大类：

*   常规的先入先出队列
*   等到队列元素集聚之后才统一安排执行的Barrier模型