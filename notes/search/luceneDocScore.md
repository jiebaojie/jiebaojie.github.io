---
layout: post
notes: true
subtitle: "【技术博客】影响Lucene对文档打分的四种方式"
comments: false
author: "刘超觉先"
date: 2017-08-30 00:00:00

---


原文：[http://www.cnblogs.com/forfuture1978/archive/2010/02/08/1666137.html](http://www.cnblogs.com/forfuture1978/archive/2010/02/08/1666137.html)

*   目录
{:toc }

# 在索引阶段设置Document Boost和Field Boost，存储在(.nrm)文件中

如果希望某些文档和某些域比其他的域更重要，如果此文档和此域包含所要查询的词则应该得分较高，则可以在索引阶段设定文档的boost和域的boost值。

这些值是在索引阶段就写入索引文件的，存储在标准化因子(.nrm)文件中，一旦设定，除非删除此文档，否则无法改变。

如果不进行设定，则Document Boost和Field Boost默认为1。

Document Boost及FieldBoost的设定方式如下：

	Document doc = new Document();
	Field f = new Field("contents", "hello world", Field.Store.NO, Field.Index.ANALYZED);
	f.setBoost(100);
	doc.add(f);
	doc.setBoost(100);
	
两者是如何影响Lucene的文档打分的呢？

让我们首先来看一下Lucene的文档打分的公式：

![](/img/notes/search/luceneDocScore/score.png)

Document Boost和Field Boost影响的是norm(t, d)，其公式如下：

![](/img/notes/search/luceneDocScore/norm.png)

它包括三个参数：

*	Document boost：此值越大，说明此文档越重要。
*	Field boost：此域越大，说明此域越重要。
*	lengthNorm(field) = (1.0 / Math.sqrt(numTerms))：一个域中包含的Term总数越多，也即文档越长，此值越小，文档越短，此值越大。

其中第三个参数可以在自己的Similarity中影响打分。

当然，也可以在添加Field的时候，设置Field.Index.ANALYZED_NO_NORMS或Field.Index.NOT_ANALYZED_NO_NORMS，完全不用norm，来节约空间。

根据Lucene的注释，No norms means that index-time field and document boosting and field length normalization are disabled.  The benefit is less memory usage as norms take up one byte of RAM per indexed field for every document in the index, during searching.  Note that once you index a given field with norms enabled, disabling norms will have no effect. 没有norms意味着索引阶段禁用了文档boost和域的boost及长度标准化。好处在于节省内存，不用在搜索阶段为索引中的每篇文档的每个域都占用一个字节来保存norms信息了。但是对norms信息的禁用是必须全部域都禁用的，一旦有一个域不禁用，则其他禁用的域也会存放默认的norms值。因为为了加快norms的搜索速度，Lucene是根据文档号乘以每篇文档的norms信息所占用的大小来计算偏移量的，中间少一篇文档，偏移量将无法计算。也即norms信息要么都保存，要么都不保存。

# 在搜索语句中，设置Query Boost

在搜索中，我们可以指定，某些词对我们来说更重要，我们可以设置这个词的boost：

	common^4 hello
	
使得包含common的文档比包含hello的文档获得更高的分数。

由于在Lucene中，一个Term定义为Field:Term，则也可以影响不同域的打分：

	title:common^4 content:common
	
使得title中包含common的文档比content中包含common的文档获得更高的分数。

# 继承并实现自己的Similarity

Similariy是计算Lucene打分的最主要的类，实现其中的很多借口可以干预打分的过程。

1.	float computeNorm(String field, FieldInvertState state)
2.	float lengthNorm(String fieldName, int numTokens)
3.	float queryNorm(float sumOfSquaredWeights)
4.	float tf(float freq)
5.	float idf(int docFreq, int numDocs)
6.	float coord(int overlap, int maxOverlap)
7.	float scorePayload(int docId, String fieldName, int start, int end, byte [] payload, int offset, int length)

它们分别影响Lucene打分计算的如下部分：

	score(q,d) = (6)coord(q,d) · (3)queryNorm(q) · ∑( (4)tf(t in d) · (5)idf(t)2 · t.getBoost() · (1)norm(t,d) )
	norm(t,d) = doc.getBoost() · (2)lengthNorm(field) · ∏f.getBoost()
	
## (1) float computeNorm(String field, FieldInvertState state)

影响标准化因子的计算，如上述，他主要包含了三部分：文档boost，域boost，以及文档长度归一化。此函数一般按照上面norm(t, d)的公式进行计算。

## (2) float lengthNorm(String fieldName, int numTokens)

主要计算文档长度的归一化，默认是1.0 / Math.sqrt(numTerms)。

因为在索引中，不同的文档长度不一样，很显然，对于任意一个term，在长的文档中的tf要大的多，因而分数也越高，这样对小的文档不公平，举一个极端的例子，在一篇1000万个词的鸿篇巨著中，"lucene"这个词出现了11次，而在一篇12个词的短小文档中，"lucene"这个词出现了10次，如果不考虑长度在内，当然鸿篇巨著应该分数更高，然而显然这篇小文档才是真正关注"lucene"的。

因而在此处是要除以文档的长度，从而减少因文档长度带来的打分不公。

然而现在这个公式是偏向于首先返回短小的文档的，这样在实际应用中使得搜索结果也很难看。

于是在实践中，要根据项目的需要，根据搜索的领域，改写lengthNorm的计算公式。比如我想做一个经济学论文的搜索系统，经过一定时间的调研，发现大多数的经济学论文的长度在8000到10000词，因而lengthNorm的公式应该是一个倒抛物线型的，8000到10000词的论文分数最高，更短或更长的分数都应该偏低，方能够返回给用户最好的数据。

## (3) float queryNorm(float sumOfSquaredWeights)

这是按照向量空间模型，对query向量的归一化。此值并不影响排序，而仅仅使得不同的query之间的分数可以比较。

## (4) float tf(float freq)

freq是指在一篇文档中包含的某个词的数目。tf是根据此数目给出的分数，默认为Math.sqrt(freq)。也即此项并不是随着包含的数目的增多而线性增加的。

## (5) float idf(int docFreq, int numDocs)

idf是根据包含某个词的文档数以及总文档数计算出的分数，默认为(Math.log(numDocs/(double)(docFreq+1)) + 1.0)。

由于此项计算涉及到总文档数和包含此词的文档数，因而需要全局的文档数信息，这给跨索引搜索造成麻烦。

## (6) float coord(int overlap, int maxOverlap)

一次搜索可能包含多个搜索词，而一篇文档中也可能包含多个搜索词，此项表示，当一篇文档中包含的搜索词越多，则此文档则打分越高。

## (7) float scorePayload(int docId, String fieldName, int start, int end, byte [] payload, int offset, int length)

由于Lucene引入了payload，因而可以存储一些自己的信息，用户可以根据自己存储的信息，来影响Lucene的打分。

payload的定义

我们知道，索引是以倒排表形式存储的，对于每一个词，都保存了包含这个词的一个链表，当然为了加快查询速度，此链表多用跳跃表进行存储。

Payload信息就是存储在倒排表中的，同文档号一起存放，多用于存储与每篇文档相关的一些信息。当然这部分信息也可以存储域里(stored Field)，两者从功能上基本是一样的，然而当要存储的信息很多的时候，存放在倒排表里，利用跳跃表，有利于大大提高搜索速度。

由payload的定义，我们可以看出，payload可以存储一些不但与文档相关，而且与查询词也相关的信息。比如某篇文档的某个词有特殊性，则可以在这个词的这个文档的position信息后存储payload信息，使得当搜索这个词的时候，这篇文档获得较高的分数。


# 继承并实现自己的collector

以上各种方法，已经把Lucene score计算公式的所有变量都涉及了，如果这还不能满足您的要求，还可以继承实现自己的collector。

在Lucene 2.4中，HitCollector有个函数public abstract void collect(int doc, float score)，用来收集搜索的结果。

此函数将docid和score插入一个PriorityQueue中，使得得分最高的文档先返回。

我们可以继承HitCollector，并在此函数中对score进行修改，然后再插入PriorityQueue，或者插入自己的数据结构。

在Lucene 3.0中，Collector接口为void collect(int doc)。