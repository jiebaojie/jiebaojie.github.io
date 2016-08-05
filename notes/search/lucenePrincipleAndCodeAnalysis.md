---
layout: post
notes: true
subtitle: Lucene原理与代码分析完整版（更新中）
comments: false
author: "觉先(forfuture1978)"
date: 2016-08-02 12:00:00
header-img: 
tags:

---

# 第一篇：原理篇

## 第一章：全文检索的基本原理

### 一、总论

Lucene是一个高效的，基于Java的全文检索库。

结构化数据和非结构化数据：

*   结构化数据：指具有固定格式或有限长度的数据,如数据库,元数据等。
*   非结构化数据：又叫全文数据。指不定长或无固定格式的数据,如邮件,word 文档等。
*   半结构化数据：如XML、HTML等，当根据需要可按结构化数据来处理，也可抽取出纯文本按非结构化数据来处理。

两种搜索：

*   对结构化数据的搜索：如对数据库的搜索,用 SQL 语句。再如对元数据的搜索,如利用windows 搜索对文件名,类型,修改时间进行搜索等。
*   对非结构化数据的搜索：如利用 windows 的搜索也可以搜索文件内容,Linux 下的 grep命令,再如用 Google 和百度可以搜索大量内容数据。

两种非结构化数据（全文数据）的搜索方法：

*   顺序扫描法（Serial Scanning）
*   全文检索：将非结构化数据中的一部分信息提取 出来,重新组织,使其变得有一定结构,然后对此有一定结构的数据进行搜索,从而达到搜 索相对较快的目的。从非结构化数据中提取出的然后重新组织的信息,我们称之索引。

![](/img/notes/search/lucenePrincipleAndCodeAnalysis/lucene.png)

全文检索的两个过程：索引创建（Indexing）和搜索索引（Search）。

*   索引创建：将现实世界中所有的结构化和非结构化数据提取信息,创建索引的过程。
*   搜索索引：就是得到用户的查询请求,搜索创建的索引,然后返回结果的过程。

全文检索三个重要问题：

1.  索引里面究竟存些什么？(Index)
2.  如何创建索引？(Indexing)
3.  如何对索引进行搜索？(Search)

### 二、索引里面究竟存些什么

反向索引：保存从字符串到文件的映射，大大提高搜索速度。

![](/img/notes/search/lucenePrincipleAndCodeAnalysis/reverse_index.png)

全文搜索相对于顺序扫描的优势之一：一次索引，多次使用。

### 三、如何创建索引

#### 第一步：一些要索引的原文档（Document）。

#### 第二步：将原文档传给分词组件（Tokenizer）。

分词组件(Tokenizer)会做以下几件事情(此过程称为 Tokenize):

1.  将文档分成一个一个单独的词。
2.  去除标点符号。
3.  去除停词(Stop word)，Stop word是一种语言中最普通的一些单词，没有特别的意义，不建议称为搜索的关键词，如英语中"the"、"a"、"this"等。

#### 第三步：将得到的词元（Token）传给语言处理组件（Linguistic Processor）。

对于英语,语言处理组件(Linguistic Processor)一般做以下几点:

1.  变为小写（Lowercase）。
2.  将单词缩减为词根形式,如“cars”到“car”等。这种操作称为:stemming。
3.  将单词转变为词根形式,如“drove”到“drive”等。这种操作称为:lemmatization。

Stemming和lemmatization的异同：

*   相同之处：Stemming和lemmatization都要使词汇称为词根形式。
*   两者的方式不同：
    *   Stemming 主要是采取某种固定的算法来做这种缩减,如去除“s”,去除“ing”加“e”, 将“ational”变为“ate”,将“tional”变为“tion”。
    *   Lemmatization 采用的是“转变”的方式:“drove”到“drove”,“driving”到“drive”。
*   两者的算法不同：
    *   Stemming 主要是采取某种固定的算法来做这种缩减,如去除“s”,去除“ing”加“e”, 将“ational”变为“ate”,将“tional”变为“tion”。
    *   Lemmatization 主要是采用保存某种字典的方式做这种转变。比如字典中有“driving” 到“drive”,“drove”到“drive”,“am, is, are”到“be”的映射,做转变时,只要查字典就 可以了。
*   Stemming 和 lemmatization 不是互斥关系,是有交集的,有的词利用这两种方式都能达到相同的转换。

语言处理组件(linguistic processor)的结果称为词(Term)。

#### 第四步：将得到的词(Term)传给索引组件(Indexer)。

索引组件(Indexer)主要做以下几件事情:

##### 1. 利用得到的词(Term)创建一个字典

##### 2. 对字典按字母顺序进行排序

##### 3. 合并相同的词(Term)成为文档倒排(Posting List)链表

![](/img/notes/search/lucenePrincipleAndCodeAnalysis/posting_list.png)

*   Document Frequency 即文档频次，表示总共有多少文件包含此词（Term）。
*   Frequency 即词频率，表示此文件中包含了几个此词（Term）。

### 四、如何对索引进行搜索？

搜索主要分为以下几步：

#### 第一步：用户输入查询语句。

#### 第二步：对查询语句进行词法分析，语法分析，及语言处理。

##### 1. 词法分析主要用来识别单词和关键字。

##### 2. 语法分析主要是根据查询语句的语法规则来形成一棵语法树。

![](/img/notes/search/lucenePrincipleAndCodeAnalysis/language_tree.jpg)

##### 3. 语言处理同索引过程中的语言处理几乎相同。

#### 第三步：搜索索引，得到符合语法树的文档。

#### 第四步：根据得到的文档和查询语句的相关性，对结果进行排序。

##### 1. 计算权重（Term weight）的过程。

影响一个词(Term)在一篇文档中的重要性主要有两个因素:

*   Term Frequency(tf)：即此Term在此文档中出现了多少次。tf越大说明越重要。
*   Document Frequency(df)：即有多少文档包含词Term。df越大说明越不重要。

![](/img/notes/search/lucenePrincipleAndCodeAnalysis/term_weight.jpg)

##### 2. 判断 Term 之间的关系从而得到文档相关性的过程,也即 向量空间模型的算法(VSM)。

![](/img/notes/search/lucenePrincipleAndCodeAnalysis/vsm.jpg)