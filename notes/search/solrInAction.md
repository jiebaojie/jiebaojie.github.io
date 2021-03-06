---
layout: post
notes: true
subtitle: Solr in Action
comments: false
author: "Trey Grainger, Timothy Potter(范炜 等译)"
date: 2017-10-29 00:00:00

---

![](/img/notes/search/solrInAction/solr_in_action.jpg)

*   目录
{:toc }

# 第1部分 初识Solr

Solr的两个最重要的功能：索引数据和执行查询。

## 1. Solr入门

本章要点：

*	搜索引擎处理的数据所具备的特征
*	常见搜索引擎用例
*	Solr核心组件
*	选择Solr的理由
*	功能概述

Solr是可扩展的、开箱即用的企业级搜索引擎，用来搜索大规模文本数据并根据相关度排序结果。

*	可扩展——Solr通过集群中多台服务器的分布式运行实例扩展
*	开箱即用——Solr是开源的，易于安装和配置，并提供预先配置好的示例服务器，方便上手
*	为搜索优化——Solr速度很快，以亚秒级速度执行复杂查询，往往只需花费几十毫秒
*	大规模文档——Solr可以用于处理包含百万级文档的索引
*	以文本为中心——Solr针对自然语言文本搜索进行优化，如电子邮件、网页、简历、PDF文档、社交消息（推文和博客）
*	根据相关度对结果排序——Solr根据文档与用户查询的相关程度对文档进行排序，并以此顺序返回结果文档
*	根据相关度对结果排序——Solr根据文档与用户查询的相关度程度对文档进行排序，并以此顺序返回结果文档

### 1.1 为什么需要搜索引擎

#### 1.1.1 管理以文本为中心的数据

类似Solr这样的搜索引擎擅长处理的数据表现为4个主要特征：

1.	以文本为中心
2.	读主导
3.	面向文档
4.	灵活的模式

#### 1.1.2 常见的搜索引擎用例

##### 基础关键词搜索

为了达到出色的用户体验，必须考虑以下几点：

*	必须快速返回相关结果，大多数情况下应该在一秒之内或更短
*	在用户拼错一些查询词时，需要进行拼写纠错
*	自动建议会减少击键次数，特别对移动应用而言
*	识别查询词项的同义词
*	匹配包含查询词项语言形式变化的文档
*	短语处理是必要的，即用户想要匹配整个短语或部分文字
*	必须处理查询中包含的"a"、"an"、"the"等常用词
*	如果用户对排名靠前的搜索结果不满意，必须能继续查看更多结果

根据相关度对文档进行排名是非常重要的，原因如下：

*	现代搜索引擎通常存储大量文档，往往以百万计或数十亿计。若部根据查询相关度对文档进行排名，就会缺乏清晰的导航方式，用户将无法应对搜索结果
*	用户更习惯于使用那些只需要输入几个关键词就能得到结果的搜索引擎，用户没有耐心，希望搜索引擎能达到“不用我说，你懂得”的结果。

第3章会深入介绍文档排名

##### 超越关键词搜索

除了返回匹配的文档之外，还应告诉用户下一步该怎么做。例如，根据文档特征对搜索结果进行分类，方便用户缩小搜索范围。这就是所谓的*分面搜索*，是Solr的强项之一（第8章）

##### 不要将搜索引擎用于......

不适合搜索引擎的用例：

*	返回大量文档
*	深度分析任务
*	SQL复杂关系型数据结构中的关系发现
*	支持文档级别的安全策略

### 1.2 Solr是什么

Solr不是什么

*	Solr不是Google或Bing这样的网络搜索引擎
*	Solr不具备网站搜索引擎优化（SEO）方面的功能

Solr的一些主要功能：

*	基础关键词搜索
*	强大的地理空间查询解决方案
*	用户执行查询后，Solr的分面功能可以对搜索结果进一步细分，显示结果集的文档特征。

#### 1.2.1 信息检索引擎

信息检索（Information Retrieval, IR）是从大规模集合（通常存储在计算机）中查找满足特定信息需求的具有非结构化性质（通常是文本）的资料（通常是文档）的过程。

第3章会介绍倒排索引的原理。

#### 1.2.2 灵活的模式管理

通过schema.xml的XML配置文件，Solr提供了一种简单明了的索引结构定义和字段表示及分析方法。Solr后台使用schema.xml表示所有可能的字段和数据类型，将文档映射为Lucene索引。这样可以节省编程时间，并让索引结果更易于理解和交流。

Solr还对Lucene核心索引功能进行了扩展。Solr增加了复制字段（copy field）和动态字段（dynamic field）。

第5章和第6章将深入介绍schema.xml。

#### 1.2.3 Java Web应用

Solr是一个Java Web应用，可以运行在任何主流Java Servlet引擎中。

#### 1.2.4 一台服务器上的多个索引

Solr支持在单个引擎上运行多个内核。

每个内核可视为一个单独的索引和配置，并且一个Solr实例可以包含多个内核。

Solr支持多核的一个用途是数据分区。

#### 1.2.5 可扩展性（插件）

每个子系统都是由模块化的“管道”构成的，通过插件方式实现新功能。

#### 1.2.6 可伸缩性

Solr提供灵活的缓存管理功能，帮助服务器重用运算量大的数据扩容（第4章介绍）

Solr伸缩行的两个最常见维度：

*	查询吞吐量
*	文档索引量

#### 1.2.7 容错性

第12章和第13章将讨论故障转移场景

Solr拥有先进的、精心设计的架构来满足可伸缩性和容错性

### 1.3 选择Solr的理由

#### 1.3.1 面向软件架构师的Solr 

软件架构师评估一项新技术时，必须考虑许多因素，包括稳定性、可伸缩性与容错性。

SolrCloud的一些新功能点：

*	集中配置
*	无单点故障（Single Point of Failure, SPoF）的分布式索引
*	故障会自动转移到新的分片代表（shard leader）
*	查询可以发送至集群中的任何一个节点，触发跨所有分片的完整的分布式搜索，内置了故障转移和负载均衡支持功能

第12章：介绍Solr的伸缩性

第13章：介绍SolrCloud的新功能

#### 1.3.2 面向系统管理员的Solr

#### 1.3.3 面向CEO的Solr

### 1.4 功能概述

从以下三个方面快速了解以下Solr的主要功能：

*	用户体验功能
*	数据建模功能
*	Solr4的新功能

#### 1.4.1 用户体验功能

可以自己动手开发一些搜索UI组件：

*	分页与排序
*	分面
*	自动建议
*	拼写检查
*	搜索结果高亮
*	地理空间搜索

##### 分页与排序

第7章介绍

##### 分面

分面对搜索结果进行归类分组，帮助用户限定查询条件和发现更多信息（第8章深入介绍）

##### 自动建议

自动建议可以减少错误查询的数量

自动建议向用户展示索引中可用的词项与短语（第10章）

##### 拼写检查

Solr的拼写检查支持以下两种基本模式：

*	自动更正
*	你要找的是不是

##### 搜索结果高亮

搜索结果高亮对越长的文档效果越好（第9章）

##### 地理空间搜索

地理位置是Solr 4的高级功能

Solr 4另一个令人兴奋的附加功能是支持地理形状的索引。

第15章介绍地理空间搜索

#### 1.4.2 数据建模功能

数据建模功能具体内容如下：

*	结果分组/字段折叠
*	灵活的查询支持
*	连接
*	文档聚类
*	导入各种文档格式，如PDF何Word
*	从关系型数据库中导入数据
*	多语种支持

##### 结果分组/字段折叠

字典折叠的典型例子是线程化的电子邮件讨论。根据特定的查询，邮件会在发起对话的第一个邮件消息下面被分组。第11章介绍结果分组/字段折叠。

##### 灵活的查询支持

Solr提供许多强大的查询功能，包括：

*	”与“、”或“、”非“条件逻辑
*	通配符匹配
*	支持日期和数字的区间查询
*	支持词项间隔距离的短语查询
*	模糊字符串匹配
*	正则表达式匹配
*	函数查询

第7章将详细介绍这些查询功能

##### 连接

第15章将讨论连接（Join）

##### 文档聚类

第16章会简要讨论聚类技术

##### 导入各种文档格式，如PDF和Word

第12章介绍各种文档格式的导入

##### 从关系型数据库中导入数据

第12章将介绍Solr的数据导入处理器（DIH）

##### 多语种支持

第14章将介绍Solr的语种检测与多语种文本分析

#### 1.4.3 Solr 4的新功能

新功能：

*	近实时搜索
*	原子更新与乐观并发
*	实时GET功能
*	使用事务日志实现写持续性
*	使用ZooKeeper实现简易分片和复制

##### 近实时搜索

Solr的近实时搜索（Near Real-Time，NRT）功能实现了文档添加到索引后的几秒钟之内，就能很快被搜索到。（第13章介绍）

##### 原子更新与乐观并发

原子更新功能允许客户端应用在已有文档上添加、更新、删除和对字段增值，而且无须重新发送整个文档。

Solr使用乐观并发机制提防不兼容的更新。

##### 实时GET功能

无论文档是否提交到索引，你都可以使用唯一标识符检索最新版本。

##### 使用事务日志实现写持续性

当文档发送到Solr进行索引时，会被写入事务日志中，以防止服务器发生故障造成数据丢失。

Solr的事务日志解除了更新可见性与更新持久性的绑定。第5章将深入讨论持久性写入与提交策略。

##### 使用ZooKeeper实现简易分片和复制

在Solr中，ZooKeeper负责指定分片代表与分片副本，并对服务请求可用的服务器进行跟踪。第13章将详细介绍SolrCloud。

### 1.5 本章小结

## 2. Solr上手

本章要点：

*	下载并安装Apache Solr 4.7
*	运行Solr的示例服务器
*	实现排序、分页和结果的格式化
*	探索Solritas的示例搜索用户界面

SolrCloud的解释及其与Solr 4的区别

SolrCloud是Solr 4功能子集的代号，让Solr服务器易于配置并运行在可扩展的、具有容错能力的集群上。SolrCloud可视为Solr 4分布式安装的一种配置方法。第13章会深入介绍SolrCloud。

### 2.1 开始上手

#### 2.1.1 Solr的安装

#### 2.1.2 启动Solr的示例服务器

Solr 4.7.0解压缩后的目录列表：

*	contrib：contrib文件夹包含扩展的源代码，例如，聚类，语种检测
*	dist：dist文件夹包含contrib模块的JAR包，例如，SolrJ客户端和Solr WAR
*	docs：docs问价夹包含contrib模块的HTML说明文档和一个Solr简明教程
*	example：example文件夹包含Solr示例服务器
*	example/solr：示例服务器的solr主目录
*	licenses：Solr所有相关依赖的许可文件

#### 2.1.3 了解Solr主目录

Solr的内核由配置文件、Lucene索引文件和Solr事务日志组成。

Solr示例服务器的默认Solr主目录。它包括一个名为collection1的单一内核，通过solr.xml进行配置。collection1的目录对应collection1的内核，包括具体的内核配置文件、Lucene索引及事务日志：

*	example/exampledocs：示例文档添加到示例索引
*	example/solr：示例默认的solr主目录
*	example/solr/collection1：collection1是一个内核
*	example/solr/collection1/conf：conf目录包含集合1内核的配置信息
*	example/solr/collection1/conf/lang：语言相关的文件，例如，停用词列表
*	example/solr/collection1/conf/schema.xml：schema.xml是对内核（如集合1）进行文本分析与索引的主要配置文件
*	example/solr/collection1/conf/solrconfig.xml：solrconfig.xml是一个内核的主要配置文件
*	example/solr/collection1/data：collection1内核的Lucene索引文件

#### 2.1.4 对示例文档进行索引

### 2.2 一切都关乎搜索

#### 2.2.1 Solr查询表单详解

#### 2.2.2 Solr的搜索返回机制

#### 2.2.3 排名检索

第3、7、16章会介绍更多有关排名检索与权重调整（boosting）的知识

#### 2.2.4 分页和排序

##### 分页

因为底层的Lucene索引并未对以此返回大量文档做出优化设计，所以尽可能使用小页面是非常重要的

##### 排序

Solr找到与查询相关的所有文档后，会对整个文档集合进行排序和分页。

#### 2.2.5 扩展的搜索功能

扩展的搜索功能：

*	dismax：析取最大查询解析器（第7章）
*	edismax：扩展的析取最大解析器（第7章）
*	h1：搜索结果高亮（第9章）
*	facet：分面（第8章）
*	spatial：地理空间搜索（第15章）
*	spellcheck：查询词项拼写检查（第10章）

Solritas界面的搜索功能：Solaritas简单示例，展示了各种搜索组件的使用方法。

### 2.3 Solr管理控制台一览

管理控制台的一些特点：

*	了解如何通过仪表盘配置Solr实例
*	从日志中查看最近的日志信息
*	从日志级别上暂时变更日志的详细设置
*	在核心管理器中添加并管理多个内核
*	查看Java的系统属性
*	通过线程转储在Java虚拟机JVM中获取所有的活动线程

内核专属页面允许执行以下操作：

*	查看内核的具体属性，例如，主要核心页面的Lucene片段数量
*	向内核发送一个快速请求，使用Ping确认系统正常运行与响应
*	在特定内核的索引上使用Query执行查询语句
*	在Schema中查看该内核当前活跃的schema.xml
*	从内核的Config中查看当前活跃的solrconfig.xml
*	在Replication中查看如何将索引复制到其他服务器上
*	在Analysis中进行文本分析测试
*	通过Schema Browser确定文档待分析的字段
*	通过Schema Browser的Local Term Info工具得到字段的热门词项信息
*	通过PlugIns/Stats查看插件的状态与配置
*	查看Solr内核缓存区域的统计信息，通过Plugins/Stats查看documentCache（文档缓存）中的文档数量
*	通过Dataimport管理DIH。

### 2.4 根据需求改造搜索示例服务器

清理索引：有时需要开启一个全新的索引。Solr停止后，通过删除内核的data/目录下的内容，移除所有文档，例如solr/collection1/data/*。重启Solr，一个不包含任何文档的全新索引就生成了。

### 2.5 本章小结

## 3. Solr基础理论 

本章要点：

*	Solr与传统数据库技术的区别
*	Solr内部索引的基本结构
*	Solr如何使用词项、短语与模糊匹配来实现复杂查询
*	Solr如何计算与查询匹配的最相关文档的得分
*	如何在返回相关的结果与尽可能多的结果之间做出权衡
*	如何对内容进行规范化文档建模
*	如何扩展Solr服务器集群，用以处理数十亿的文档和查询

### 3.1 搜索、匹配与找寻内容

搜索引擎，尤其是Solr，致力于解决一类特定的问题：搜索大量非结构化文本，并返回最相关的搜索结果。

#### 3.1.1 何为文档

提交给Solr处理的每一份数据都是一个文档。文档可以是一篇新闻报道、一份简历、社交用户信息，甚至是一整本书。每个文档包含一个或多个字段

当首次接收到包含新字段的文档时，Solr会自动猜测未知的新字段类型。通过检查字段中的数据类型，自动将字段增加到Solr的Schema中。如果输入的数据难以理解，Solr可能会对字段类型识别失败，因此，更好的做法是预先定义好字段。

#### 3.1.2 基本搜索问题

查询执行中存在一些困难：

*	仅执行子字符串匹配，不能区分出词
*	不能理解语言变体，例如“buying”与“buy”。
*	不能理解同义词，例如“buying”与“purchasing”、“home“与”house“。
*	类似”a“这样不重要的词汇会影响到预期的搜索结果（相关结果被排除在外或不相关结果的出现，取决于这些词是”全部“还是”任意“匹配）。
*	结果的相关度排序是无意义的。仅匹配到一个查询词的图书比匹配到多个查询词的图书排名更靠前。

Solr会对内容和查询进行文本分析，确定文本相似的词，理解并匹配同义词，移除”a“、”the“和”of“这类不重要的词，每个搜索结果的得分是基于它与查询词的匹配程度来计算的，以确保最佳结果排在前面，用户无须为找寻期望的内容而翻阅大量不相关的页面结果。Solr之所以能完成以上工作，是因为使用了索引将内容映射至文档的方式。这与传统数据库模型——文档映射至内容的方式不同。倒排索引是搜索引擎运作的核心。

#### 3.1.3 倒排索引

倒排索引的两个重要细节：

*	倒排索引中的所有词项对应一个或多个文档
*	倒排索引中的词项是根据字典顺序升序排列

#### 3.1.4 词项、短语与布尔逻辑

三种匹配需求：

*	搜索两个不同的词项，new和house，要求两者都被匹配
*	搜索两个不同的词项，new和house，要求匹配其一即可
*	搜索短语”new house“，要求精确匹配。

##### 必备词项

AND

##### 可选词项

OR

##### 排除词项

NOT

##### 短语

Solr不仅支持单个词项搜索，还支持短语搜索，确保多个词项按特定的顺序出现。

##### 组合表达式

Solr查询语法使用括号组合词项的方式来构造任意复杂的查询表达式

#### 3.1.5 找到文档集

一旦形成每个词项的匹配文档列表之后，Lucene就会执行集合操作，得到与该查询匹配的适合的结果集。

#### 3.1.6 短语查询与术语位置

词项位置：记录词项在文档中的相对位置（可选项）。

#### 3.1.7 模糊匹配

模糊匹配可以对索引中的词项进行并不那么精确的匹配。

*	通配符搜索：如想搜索特定前缀开头的查询
*	模糊搜索或编辑距离搜索：如想搜索一个或两个字符的拼写变化
*	邻近搜索：如根据特定距离来匹配两个查询词

##### 通配符搜索

执行首位通配符查询是一项花销甚大的操作

通配符只适用于单个查询词，不适用短语搜索

##### 区间搜索

Solr 还提供了在已知区间值中进行搜索的功能，适用于在一个区间内搜索特定文档子集。

##### 模糊/编辑距离搜索

Solr 基于 Damerau-Levenshtein 距离的编辑距离度量方法提供了字符变体的处理手段，可以有效解决80%以上的认为拼写错误。

Solr 使用~符号表示模糊编辑距离搜索。

查询：administrator~N 匹配N个以内的编辑距离

注意：两个以上的编辑距离会使得搜索速度大幅下降，也可能匹配出意外的词项。一到两个编辑距离的术语搜索使用有效的Levenshtein自动机方法执行，但超过两个编辑距离的查询就会退回到更慢的编辑距离实现方法。

## 16. 精通相关度

### 16.5 个性化搜索与推荐

#### 16.5.1 推荐 vs. 推荐

搜索引擎与推荐引擎的工作原理相同：通过建立文档之间链接的稀疏矩阵，使用某种相似性度量来寻找最佳匹配。搜索是“词项——文档”矩阵，推荐是“偏好——文档”矩阵。

搜索与推荐的真正区别在于：

*	搜索一般是需要用户输入的手工任务
*	推荐通常是了解用户的有关情况

推荐是一种自动化搜索，非人工判断哪些是用户想要的相关内容

基于内容的推荐方法：

*	基于属性的匹配
*	基于层级分类的匹配
*	基于抽取的兴趣物品的匹配（更多类似结果）
*	基于概念的匹配
*	基于地理位置的匹配

协同过滤技术：基于用户与内容的交互进行推荐，让Solr从用户行为中不断学习，通过返回的更多相关结果反映其智能化程度。

#### 16.5.2 基于属性的匹配

q=(jobtitle:"nurse educator"^25 OR jobtitle:(nurse educator)^10) AND ((city:"Boston" AND state:"MA")^15 OR state:"MA") AND \_val\_:"map(salary, 40000, 60000, 10, 0)"

#### 16.5.3 分层匹配

假设对用户和内容进行分类，将其归入某种层级中（从一般类目到具体类目），这样就可以对这个层级进行查询，对更加专指的匹配赋予更高的相关度权重。

df=classification&q=(("healthcare.nursing.oncology"^40 OR "healthcare.nursing"^20 OR "healthcare"^10) OR ("healthcare.nursing.transplant"^20 OR "healthcare.nursing"^10 OR "healthcare"^5) OR ("education.postsecondary.nursing"^10 OR "education.postsecondary"^5 OR "education"))

#### 16.5.4 更多类似结果

Sorl的更多类似结果处理器能够处理任何文档，它从文档中抽取感兴趣的词项，使用这些词项进行关键词搜索，以此寻找类似的文档。从内部看，更多类似结果处理器会从文档中抽取感兴趣的词项，将文档视为词项向量，根据tf-idf相似度计算，抽取匹配度最高的词项。

##### 外部文档的更多类似结果

更多类似结果处理器还可以根据传入外部文档的全文进行推荐。

#### 16.5.5 基于概念的匹配

Solr的聚类组件

#### 16.5.6 地理位置的匹配

谨慎地考虑是否要为高度敏感的用户设置严格的位置过滤器（15.2）

为地理位置接近的文档略微提升相关度（16.3.4）

#### 16.5.7 协同过滤

协同过滤不是基于内容相似度，而是基于用户与文档的交互行为。