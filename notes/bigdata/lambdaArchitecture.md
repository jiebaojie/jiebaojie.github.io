---
layout: post
notes: true
subtitle: "【技术博客】用于实时大数据处理的Lambda架构"
comments: false
author: "可口可乐"
date: 2017-04-05 00:00:00

---


原文：[https://blog.csdn.net/brucesea/article/details/45937875](https://blog.csdn.net/brucesea/article/details/45937875)

*   目录
{:toc }

# 1. Lambda架构背景介绍

Lambda架构是由Storm的作者Nathan Marz提出的一个实时大数据处理框架。Marz在Twitter工作期间开发了著名的实时大数据处理框架Storm，Lambda架构是其根据多年进行分布式大数据系统的经验总结提炼而成。

Lambda架构的目标是设计出一个能满足实时大数据系统关键特性的架构，包括有：高容错、低延时和可扩展等。Lambda架构整合离线计算和实时计算，融合不可变性（Immunability），读写分离和复杂性隔离等一系列架构原则，可集成Hadoop，Kafka，Storm，Spark，Hbase等各类大数据组件。

# 2. 大数据系统的关键特性

Marz认为大数据系统应具有以下的关键特性：

*	Robust and fault-tolerant（容错性和鲁棒性）：对大规模分布式系统来说，机器是不可靠的，可能会当机，但是系统需要是健壮、行为正确的，即使是遇到机器错误。除了机器错误，人更可能会犯错误。在软件开发中难免会有一些Bug，系统必须对有Bug的程序写入的错误数据有足够的适应能力，所以比机器容错性更加重要的容错性是人为操作容错性。对于大规模的分布式系统来说，人和机器的错误每天都可能会发生，如何应对人和机器的错误，让系统能够从错误中快速恢复尤其重要。
*	Low latency reads and updates（低延时）：很多应用对于读和写操作的延时要求非常高，要求对更新和查询的响应是低延时的。
*	Scalable（横向扩容）：当数据量/负载增大时，可扩展性的系统通过增加更多的机器资源来维持性能。也就是常说的系统需要线性可扩展，通常采用scale out（通过增加机器的个数）而不是scale up（通过增强机器的性能）。
*	General（通用性）：系统需要能够适应广泛的应用，包括金融领域、社交网络、电子商务数据分析等。
*	Extensible（可扩展）：需要增加新功能、新特性时，可扩展的系统能以最小的开发代价来增加新功能。
*	Allows ad hoc queries（方便查询）：数据中蕴含有价值，需要能够方便、快速的查询出所需要的数据。
*	Minimal maintenance（易于维护）：系统要想做到易于维护，其关键是控制其复杂性，越是复杂的系统越容易出错、越难维护。
*	Debuggable（易调试）：当出问题时，系统需要有足够的信息来调试错误，找到问题的根源。其关键是能够追根溯源到每个数据生成点。

# 3. 数据系统的本质

为了设计出能满足前述的大数据关键特性的系统，我们需要对数据系统有本质性的理解。我们可将数据系统简化为：

	数据系统 = 数据 + 查询
	
从而从数据和查询两方面来认识大数据系统的本质。

## 3.1 数据的本质

### 3.1.1 数据的特性：When & What

我们先从“数据”的特性谈起。数据是一个不可分割的单位，数据有两个关键的性质：When和What。

*	When是指数据是与时间相关的，数据一定是在某个时间点产生的。比如Log日志就隐含着按照时间先后顺序产生的数据，Log前面的日志数据一定先于Log后面的日志数据产生；消息系统中消息的接受者一定是在消息的发送者发送消息后接收到的消息。相比于数据库，数据库中表的记录就丢失了时间先后顺序的信息，中间某条记录可能是在最后一条记录产生后发生更新的。对于分布式系统，数据的时间特性尤其重要。分布式系统中数据可能产生于不同的系统中，时间决定了数据发生的全局先后顺序。比如对一个值做算术运算，先+2，后*3，与先*3，后+2，得到的结果完全不同。数据的时间性质决定了数据的全局发生先后，也就决定了数据的结果。
*	What是指数据的本身。由于数据跟某个时间点相关，所以数据的本身是不可变的(immutable)，过往的数据已经成为事实（Fact），你不可能回到过去的某个时间点去改变数据事实。这也就意味着对数据的操作其实只有两种：读取已存在的数据和添加更多的新数据。采用数据库的记法，CRUD就变成了CR，Update和Delete本质上其实是新产生的数据信息，用C来记录。

### 3.1.2 数据的存储：Store Everything Rawly and Immutably

根据上述对数据本质特性的分析，Lamba架构中对数据的存储采用的方式是：数据不可变，存储所有数据。

通过采用不可变方式存储所有的数据，可以有如下好处：

*	简单。采用不可变的数据模型，存储数据时只需要简单的往主数据集后追加数据即可。相比于采用可变的数据模型，为了Update操作，数据通常需要被索引，从而能快速找到要更新的数据去做更新操作。
*	应对人为和机器的错误。前述中提到人和机器每天都可能会出错，如何应对人和机器的错误，让系统能够从错误中快速恢复极其重要。不可变性（Immutability）和重新计算（Recomputation）则是应对人为和机器错误的常用方法。采用可变数据模型，引发错误的数据有可能被覆盖而丢失。相比于采用不可变的数据模型，因为所有的数据都在，引发错误的数据也在。修复的方法就可以简单的是遍历数据集上存储的所有的数据，丢弃错误的数据，重新计算得到Views。重新计算的关键点在于利用数据的时间特性决定的全局次序，依次顺序重新执行，必然能得到正确的结果。

当前业界有很多采用不可变数据模型来存储所有数据的例子。比如分布式数据库Datomic，基于不可变数据模型来存储数据，从而简化了设计。分布式消息中间件Kafka，基于Log日志，以追加append-only的方式来存储消息。

## 3.2 查询

查询是个什么概念？Marz给查询如下一个简单的定义：

	Query = Function(All Data)
	
该等式的含义是：查询是应用于数据集上的函数。该定义看似简单，却几乎囊括了数据库和数据系统的所有领域：RDBMS、索引、OLAP、OLTP、MapReduce、EFL、分布式文件系统、NoSQL等都可以用这个等式来表示。

让我们进一步深入看一下函数的特性，从而挖掘函数自身的特点来执行查询。

有一类称为Monoid特性的函数应用非常广泛。Monoid的概念来源于范畴学（Category Theory），其一个重要特性是满足结合律。如整数的加法就满足Monoid特性：

	(a+b)+c=a+(b+c)
	
不满足Monoid特性的函数很多时候可以转化成多个满足Monoid特性的函数的运算。如多个数的平均值Avg函数，多个平均值没法直接通过结合来得到最终的平均值，但是可以拆成分母除以分子，分母和分子都是整数的加法，从而满足Monoid特性。

Monoid的结合律特性在分布式计算中极其重要，满足Monoid特性意味着我们可以将计算分解到多台机器并行运算，然后再结合各自的部分运算结果得到最终结果。同时也意味着部分运算结果可以储存下来被别的运算共享利用（如果该运算也包含相同的部分子运算），从而减少重复运算的工作量。 

![](/img/notes/bigdata/lambdaArchitecture/monoid)

# 4. Lambda架构

有了上面对数据系统本质的探讨，下面我们来讨论大数据系统的关键问题：如何实时地在任意大数据集上进行查询？大数据再加上实时计算，问题的难度比较大。

最简单的方法是，根据前述的查询等式Query = Function(All Data)，在全体数据集上在线运行查询函数得到结果。但如果数据量比较大，该方法的计算代价太大了，所以不现实。

Lambda架构通过分解的三层架构来解决该问题：Batch Layer，Speed Layer和Serving Layer。

![](/img/notes/bigdata/lambdaArchitecture/lambda)

## 4.1 Batch Layer

Batch Layer的功能主要有两点：

*	存储数据集
*	在数据集上预先计算查询函数，构建查询所对应的View

### 4.1.1 存储数据集

根据前述对数据When&What特性的讨论，Batch Layer采用不可变模型存储所有的数据。因为数据量比较大，可以采用HDFS之类的大数据储存方案。如果需要按照数据产生的时间先后顺序存放数据，可以考虑如InfluxDB之类的时间序列数据库（TSDB）存储方案。

### 4.1.2 构建查询View

上面说到根据等式Query = Function(All Data)，在全体数据集上在线运行查询函数得到结果的代价太大。但如果我们预先在数据集上计算并保存查询函数的结果，查询的时候就可以直接返回结果（或通过简单的加工运算就可得到结果）而无需重新进行完整费时的计算了。这儿可以把Batch Layer看成是一个数据预处理的过程。我们把针对查询预先计算并保存的结果称为View，View是Lamba架构的一个核心概念，它是针对查询的优化，通过View即可以快速得到查询结果。 

![](/img/notes/bigdata/lambdaArchitecture/batch_view)

如果采用HDFS来储存数据，我们就可以使用MapReduce来在数据集上构建查询的View。Batch Layer的工作可以简单的用如下伪码表示： 

	while(true){
		recomputeBatchView();
	}
	
该工作看似简单，实质非常强大。任何人为或机器发生的错误，都可以通过修正错误后重新计算来恢复得到正确结果。

#### 对View的理解

View是一个和业务关联性比较大的概念，View的创建需要从业务自身的需求出发。一个通用的数据库查询系统，查询对应的函数千变万化，不可能穷举。但是如果从业务自身的需求出发，可以发现业务所需要的查询常常是有限的。Batch Layer需要做的一件重要的工作就是根据业务的需求，考察可能需要的各种查询，根据查询定义其在数据集上对应的Views。

## 4.2 Speed Layer

Batch Layer可以很好的处理离线数据，但有很多场景数据不断实时生成，并且需要实时查询处理。Speed Layer正是用来处理增量的实时数据。

Speed Layer和Batch Layer比较类似，对数据进行计算并生成Realtime View，其主要区别在于：

*	Speed Layer处理的数据是最近的增量数据流，Batch Layer处理的全体数据集
*	Speed Layer为了效率，接收到新数据时不断更新Realtime View，而Batch Layer根据全体离线数据集直接得到Batch View。

Lambda架构将数据处理分解为Batch Layer和Speed Layer有如下优点：

*	容错性。Speed Layer中处理的数据也不断写入Batch Layer，当Batch Layer中重新计算的数据集包含Speed Layer处理的数据集后，当前的Realtime View就可以丢弃，这也就意味着Speed Layer处理中引入的错误，在Batch Layer重新计算时都可以得到修正。这点也可以看成是CAP理论中的最终一致性（Eventual Consistency）的体现。 

![](/img/notes/bigdata/lambdaArchitecture/eventual_consistency)

*	复杂性隔离。Batch Layer处理的是离线数据，可以很好的掌控。Speed Layer采用增量算法处理实时数据，复杂性比Batch Layer要高很多。通过分开Batch Layer和Speed Layer，把复杂性隔离到Speed Layer，可以很好的提高整个系统的鲁棒性和可靠性。

## 4.3 Serving Layer

Lambda架构的Serving Layer用于响应用户的查询请求，合并Batch View和Realtime View中的结果数据集到最终的数据集。

这儿涉及到数据如何合并的问题。前面我们讨论了查询函数的Monoid性质，如果查询函数满足Monoid性质，即满足结合率，只需要简单的合并Batch View和Realtime View中的结果数据集即可。否则的话，可以把查询函数转换成多个满足Monoid性质的查询函数的运算，单独对每个满足Monoid性质的查询函数进行Batch View和Realtime View中的结果数据集合并，然后再计算得到最终的结果数据集。另外也可以根据业务自身的特性，运用业务自身的规则来对Batch View和Realtime View中的结果数据集合并。 

![](/img/notes/bigdata/lambdaArchitecture/serving_layer)

# 5. Big Picture

上面分别讨论了Lambda架构的三层：Batch Layer，Speed Layer和Serving Layer。下图给出了Lambda架构的一个完整视图和流程。

![](/img/notes/bigdata/lambdaArchitecture/big_picture)

数据流进入系统后，同时发往Batch Layer和Speed Layer处理。Batch Layer以不可变模型离线存储所有数据集，通过在全体数据集上不断重新计算构建查询所对应的Batch Views。Speed Layer处理增量的实时数据流，不断更新查询所对应的Realtime Views。Serving Layer响应用户的查询请求，合并Batch View和Realtime View中的结果数据集到最终的数据集。

## 5.1 Lambda架构组件选型

下图给出了Lambda架构中各个层常用的组件。数据流存储可选用基于不可变日志的分布式消息系统Kafka；Batch Layer数据集的存储可选用Hadoop的HDFS，或者是阿里云的ODPS；Batch View的预计算可以选用MapReduce或Spark；Batch View自身结果数据的存储可使用MySQL（查询少量的最近结果数据），或HBase（查询大量的历史结果数据）。Speed Layer增量数据的处理可选用Storm或Spark Streaming；Realtime View增量结果数据集为了满足实时更新的效率，可选用Redis等内存NoSQL。

![](/img/notes/bigdata/lambdaArchitecture/model)

## 5.2 Lambda架构组件选型原则

Lambda架构是个通用框架，各个层选型时不要局限时上面给出的组件，特别是对于View的选型。从我对Lambda架构的实践来看，因为View是个和业务关联性非常大的概念，View选择组件时关键是要根据业务的需求，来选择最适合查询的组件。不同的View组件的选择要深入挖掘数据和计算自身的特点，从而选择出最适合数据和计算自身特点的组件，同时不同的View可以选择不同的组件。

# 6. Lambda架构 vs. Event Sourcing vs. CQRS

在Lambda架构身上可以看到很多现有设计思想和架构的影子，如Event Sourcing和CQRS，这儿我们把它们和Lambda架构做一结合对比，从而去更深入的理解Lambda架构。

## 6.1 事件溯源（Event Sourcing）vs. Lambda架构

事件溯源（Event Sourcing）是由大名鼎鼎的Martin Flower大叔提出来的架构模式。Event Sourcing本质上是一种数据持久化的方式，它将引发变化的事件（Event）本身存储下来。相比于传统数据是持久化方式，存储的是事件引发的结果，而非事件本身，这样我们在保存结果的同时，实际上失去了追溯导致结果原因的机会。

这儿可以看到Lambda架构中数据集的存储和Event Sourcing中的思想是完全一致的，本质都是采用不可变的数据模型存储引发变化的事件而非变化产生的结果。从而在发生错误的时候，能够追本溯源，找到发生错误的根源，通过重新计算丢弃错误的信息来恢复系统，达到系统的容错性。

## 6.2 CQRS vs. Lambda架构

CQRS (Command Query Responsibility Segregation)将对数据的修改操作和查询操作分离，其本质和Lambda架构一样，也是一种形式的读写分离。在Lambda架构中，数据以不可变的方式存储下来（写操作），转换成查询所对应的Views，查询从View中直接得到结果数据（读操作）。

读写分离将读和写两个视角进行分离，带来的好处是复杂性的隔离，从而简化系统的设计。相比于传统做法中的将读和写操作放在一起的处理方式，对于读写操作业务非常复杂的系统，只会使系统变得异常复杂，难以维护。 

![](/img/notes/bigdata/lambdaArchitecture/read_write)

# 7. 总结

本文介绍了Lambda架构的基本概念。Lambda架构通过对数据和查询的本质认识，融合了不可变性（Immunability），读写分离和复杂性隔离等一系列架构原则，将大数据处理系统划分为Batch Layer, Speed Layer和Serving Layer三层，从而设计出一个能满足实时大数据系统关键特性（如高容错、低延时和可扩展等）的架构。Lambda架构作为一个通用的大数据处理框架，可以很方便的集成Hadoop，Kafka，Storm，Spark，Hbase等各类大数据组件。