---
layout: post
notes: true
subtitle: Lucene原理与代码分析完整版（更新中）
comments: false
author: "觉先(forfuture1978)"
date: 2016-08-02 12:00:00

---

原文地址：[Lucene原理与代码分析完整版](http://www.cnblogs.com/forfuture1978/archive/2010/06/13/1757479.html)

*   目录
{:toc }

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

### 总结

![](/img/notes/search/lucenePrincipleAndCodeAnalysis/lucene_framework.jpg)

#### 1. 索引过程：

*   有一系列被索引文件
*   被索引文件经过语法分析和语言处理形成一系列词（Term）
*   经过索引创建形成词典和反向索引表
*   通过索引存储将索引写入硬盘

#### 2. 搜索过程：

*   用户输入查询语句
*   对查询语句经过语法分析和语言分析得到一系列词（Term）。
*   通过语法分析得到一个查询树。
*   通过索引存储将索引读入到内存。
*   利用查询树搜索索引，从而得到每个词（Term）的文档链表，对文档链表进行交、差、并得到结果文档。
*   将搜索到的结果文档对查询的相关性进行排序。
*   返回查询结果给用户。

## 第二章：Lucene的总体架构

Lucene总的来说是：

*   一个高效的，可扩展的，全文检索库
*   全部用Java实现，无须配置
*   仅支持纯文本文件的索引(Indexing)和搜索(Search)
*   不负责由其他格式的文件抽取纯文本文件，或从网络中抓取文件的过程。

Lucene的架构和过程：

![](/img/notes/search/lucenePrincipleAndCodeAnalysis/lucene.png)

Lucene有索引和搜索两个过程，包含索引创建，索引，搜索三个要点。

![](/img/notes/search/lucenePrincipleAndCodeAnalysis/lucene_process.jpg)

被索引的文档用Document对象表示：

*   IndexWriter通过函数addDocument将文档添加到索引中，实现创建索引的过程。
*   Lucene的索引是应用反向索引。
*   当用户有请求时，Query代表用户的查询语句。
*   IndexSearcher通过函数search搜索Lucene Index。
*   IndexSearcher计算term weight和score并且将结果返回给用户。
*   返回给用户的文档集合用TopDocsCollector表示。

![](/img/notes/search/lucenePrincipleAndCodeAnalysis/lucenesrc.png)

索引过程如下：

*   创建一个 IndexWriter 用来写索引文件,它有几个参数,INDEX_DIR 就是索引文件所存放的位置,Analyzer 便是用来对文档进行词法分析和语言处理的。
*   创建一个 Document 代表我们要索引的文档。
*   将不同的 Field 加入到文档中。我们知道,一篇文档有多种信息,如题目,作者,修改时间,内容等。不同类型的信息用不同的 Field 来表示,在本例子中,一共有 两类信息进行了索引,一个是文件路径,一个是文件内容。其中 FileReader 的 SRC_FILE 就表示要索引的源文件。
*   IndexWriter 调用函数 addDocument 将索引写到索引文件夹中。

搜索过程如下：

*   IndexReader将磁盘上的索引信息读入到内存,INDEX_DIR 就是索引文件存放的位置。
*   创建IndexSearcher准备进行搜索。
*   创建 Analyzer 用来对查询语句进行词法分析和语言处理。
*   创建 QueryParser 用来对查询语句进行语法分析。
*   QueryParser 调用 parser 进行语法分析,形成查询语法树,放到 Query 中。
*   IndexSearcher 调用 search 对查询语法树 Query 进行搜索,得到结果TopScoreDocCollector。

![](/img/notes/search/lucenePrincipleAndCodeAnalysis/lucene_framework.jpg)

*   Lucene的analysis模块主要负责词法分析及语言处理而形成Term。
*   Lucene的index模块主要负责索引的创建,里面有IndexWriter。
*   Lucene的store模块主要负责索引的读写。
*   Lucene的QueryParser主要负责语法分析。
*   Lucene的search模块主要负责对索引的搜索。
*   Lucene的similarity模块主要负责对相关性打分的实现。

# 第二篇：代码分析篇

## 第三章：Lucene的索引文件格式

*   Lucene 的索引过程,就是按照全文检索的基本过程,将倒排表写成此文件格式的过程。
*   Lucene 的搜索过程,就是按照此文件格式将索引进去的信息读出来,然后计算每篇文档打分(score)的过程。

### 一、基本概念

![](/img/notes/search/lucenePrincipleAndCodeAnalysis/lucene_index.png)

索引（Index）：

*   在 Lucene 中一个索引是放在一个文件夹中的。
*   同一文件夹中的所有的文件构成一个 Lucene 索引。

段（Segment）：

*   一个索引可以包含多个段，段与段之间是独立的，添加新文档可以生成新的段，不同的段可以合并。
*   具有相同前缀文件的属于同一个段。
*   segments.gen 和 segments_5 是段的元数据文件，也即它们保存了段的属性信息

文档（Document）：

*   文档是我们建索引的基本单位,不同的文档是保存在不同的段中的,一个段可以包含多篇文档。
*   新添加的文档是单独保存在一个新生成的段中,随着段的合并,不同的文档合并到同一个段中。

域（Field）：

*   一篇文档包含不同类型的信息，可以分开索引，比如标题，时间，正文，作者等，都可以保存在不同的域里。
*   不同域的索引方式可以不同

词（Term）：

*   词是索引的最小单位，是经过词法分析和语言处理后的字符串。

Lucene的索引结构中，即保存了正向信息，也保存了反向信息。

所谓正向信息：

*   按层次保存了从索引，一直到词的包含关系：索引（Index）-> 段（segment）-> 文档（Document） -> 域（Field）-> 词（Term）
*   每个索引包含了哪些段，每个段包含了哪些文档，每个文档包含了哪些域，每个域包含了哪些词。
*   每个层次都保存了本层次的信息以及下一层次的元信息。

所谓反向信息：

*   保存了词典到倒排表的映射：词（Term）-> 文档（Document）

包含的反向信息的文件有：

*   XXX.tis，XXX.tii保存了词典（Term Dictionary），也即此段包含的所有的词按字典顺序的排序。
*   XXX.frq保存了倒排表，也即包含每个词的文档ID列表。
*   XXX.prx保存了倒排表中每个词在包含此词在文档中的位置。

### 二、基本类型

*   Byte：最基本的类型，长8位(bit)。
*   UInt32：由4个Byte组成。
*   UInt64：由8个Byte组成。
*   VInt64：变长的整数类型，它可能包含多个Byte，对于每个Byte的8位，其中后7位表示数值，最高1位表示是否还有另一个Byte，0表示没有，1表示有。
*   Chars：是UTF-8编码的一系列Byte。
*   String：一个字符串。首先是一个VInt来表示此字符串包含的字符的个数，接着便是UTF-8编码的字符序列Chars。

### 三、基本规则

#### 1. 前缀后缀规则（Prefix+Suffix）

所谓前缀后缀规则,即当某个词和前一个词有共同的前缀的时候,后面的词仅仅保存前缀在词中的偏移(offset),以及除前缀以外的字符串(称为后缀)。

#### 2. 差值规则（Delta）

所谓差值规则(Delta)就是先后保存两个整数的时候，后面的整数仅仅保存和前面整数的差即可。

#### 3. 或然跟随规则（A,B?）

Lucene的索引结构中存在这样的情况，某个值 A 后面可能存在某个值 B，也可能不存在，需要一个标志来表示后面是否跟随着 B。在 Lucene 中，采取以下的方式：A 的值左移一位,空出最后一位，作为标志位，来表示后面是否跟随 B，所以在这种情况下，A/2 是真正的 A 原来的值。

#### 4. 跳跃表规则（Skip list）

![](/img/notes/search/lucenePrincipleAndCodeAnalysis/skip_list.jpg)

### 四、具体格式

#### 4.1 正向信息

Index –> Segments (segments.gen, segments_N) –> Field(fnm, fdx, fdt) –> Term (tvx, tvd, tvf)

##### 4.1.1 段的元数据信息（segments_N）

一个索引(Index)可以同时存在多个 segments_N，然而当我们要打开一个索引的时候，我们必须要选择一个来打开，采取过程如下：

*   其一，在所有的 segments_N 中选择 N 最大的一个。在所有以 segments 开头，并且不是 segments.gen 的文件中,选择 N 最大的一个作为 genA。
*   其二，打开 segments.gen，其中保存了当前的 N 值。其格式如下，读出版本号(Version), 然后再读出两个 N，如果两者相等，则作为 genB。
*   其三，在上述得到的genA和genB中选择最大的那个作为当前的N，方才打开segments_N 文件。

![](/img/notes/search/lucenePrincipleAndCodeAnalysis/segments_N.gif)

Format：

*   索引文件格式的版本号。
*   由于 Lucene 是在不断开发过程中的，因而不同版本的 Lucene，其索引文件格式也不尽相同，于是规定一个版本号。
*   Lucene 2.1 此值-3，Lucene 2.9 时，此值为-9。
*   当用某个版本号的 IndexReader 读取另一个版本号生成的索引的时候，会因为此值不同而报错。

Version：

*   索引的版本号，其初始值大多数情况下从索引文件里面读出，仅仅在索引开始创建的时候，被赋予当前的时间，取得一个唯一值。

NameCount：

*   是下一个新段(Segment)的段名。
*   所有属于同一个段的索引文件都以段名作为文件名，一般为_0.xxx, _0.yyy, _1.xxx, _1.yyy ......
*   新生成的段的段名一般为原有最大段名加一。

SegCount：

*   段（Segment）的个数。

SegCount个段的元数据信息：

*   SegName
    *   段名，所有属于同一个段的文件都有以段名作为文件名。
    *   第一个段的段名为"_0"，第二个段的段名为"_1"
*   SegSize
    *   此段中包含的文档数
    *   然而此文档数是包括已经删除，又没有 optimize 的文档的，因为在 optimize
之前，Lucene 的段中包􏰀了所有被索引过的文档，而被删除的文档是保存在.del 文件中的，在搜索的过程中，是先从段中读到了被删除的文档，然后再用.del中的标志,将这篇文档过滤掉。
*   DelGen
    *   .del文件的版本号
    *   Lucene 中，在 optimize 之前，删除的文档是保存在.del 文件中的。
    *   DelGen 是每当 IndexWriter 向索引文件中提交删除操作的时候，加 1，并生成 新的.del 文件。
*   DocStoreOffset
*   DocStoreSegment
*   DocStoresCompoundFile
    *   对于域(Stored Field)和词向量(Term Vector)的存储可以有不同的方式，即可以每个段(Segment)单独存储自己的域和词向量信息，也可以多个段共享域和词向量，把它们存储到一个段中去。
    *   如果 DocStoreOffset 为-1，则此段单独存储自己的域和词向量。DocStoreSegment 和 DocStoreIsCompoundFile 在此处不被保存。
    *   如果 DocStoreOffset 不为-1，则 DocStoreSegment 保存了共享的段的名字，DocStoreOffset 则为此段的域及词向量信息在共享段中的偏移量。
*   HasSingleNormFile
    *   在搜索的过程中，标准化因子(Normalization Factor)会影响文档最后的评分。
    *   不同的文档重要性不同，不同的域重要性也不同。因而每个文档的每个域都可以有自己的标准化因子。
    *   如果 HasSingleNormFile 为 1，则所有的标准化因子都是存在.nrm 文件中的。
    *   如果 HasSingleNormFile 不是 1，则每个域都有自己的标准化因子文件.fN
*   NumField
    *   域的数量
*   NormGen
    *   如果每个域有自己的标准化因子文件，则此数组描述了每个标准化因子文件的版本号，也即.fN 的 N。
*   isCompoundFile
    *   是否保存为复合文件，也即把同一个段中的文件按照一定格式，保存在一个文件当中，这样可以减少每次打开文件的个数。
*   DeletionCount
    *   记录了此段中删除的文档的数目。
*   HasProx
    *   如果至少有一个段 omitTf 为 false，也即词频(term freqency)需要被保存，则HasProx 为 1,否则为 0。
*   Diagnostics
    *   调试信息。

User map data

*   保存了用户从字符串到字符串的映射 Map<String,String>

CheckSum

*   此文件 segment_N 的校验和。

##### 4.1.2 域(Field)的元数据信息(.fnm)

一个段(Segment)包􏰀多个域，每个域都有一些元数据信息，保存在.fnm 文件中，.fnm 文件 的格式如下:

![](/img/notes/search/lucenePrincipleAndCodeAnalysis/field_meta.png)

FNMVersion：

*   是fnm文件的版本号，对于 Lucene 2.9 为-2。

FieldsCount

*   域的数目

一个数组的域（Fields）

*   FieldName：域名，如"title"，"modified"，"content"等。
*   FieldBits：一系列标志位，表明对此域的索引方式
    *   最低位：1 表示此域被索引，0 则不被索引。所谓被索引，也即放到倒排表中去。
    *   倒数第二位：1 表示保存词向量，0 为不保存词向量。
    *   倒数第三位：1 表示在词向量中保存位置信息。
    *   倒数第四位：1 表示在词向量中保存偏移量信息。
    *   倒数第五位：1 表示不保存标准化因子
    *   倒数第六位：是否保存 payload

位置(Position)和偏移量(Offset)的区􏰁别：

*   位置是基于词 Term 的，偏移量是基于字母或汉字的。

![](/img/notes/search/lucenePrincipleAndCodeAnalysis/position_offset.png)

索引域(Indexed)和存储域(Stored)的区􏰁别：

*   一个域为什么会被存储(store)而不被索引(Index)呢?在一个文档中的所有信息中，有这样一部分信息，可能不想被索引从而可以搜索到，但是当这个文档由于其他的信息被搜索到时，可以同其他信息一同返回。

##### 4.1.3 域(Field)的数据信息(.fdt,.fdx)

![](/img/notes/search/lucenePrincipleAndCodeAnalysis/field_data.png) 

域数据文件（fdt）：

*   真正保存存储域(stored field)信息的是fdt文件
*   在一个段(segment)中总共有 segment size 篇文档，所以 fdt 文件中共有 segment size个项，每一项保存一篇文档的域的信息
*   对于每一篇文档，一开始是一个 fieldcount，也即此文档包􏰁的域的数目，接下来是 fieldcount 个项，每一项保存一个域的信息。
*   对于每一个域，fieldnum 是域号，接着是一个 8 位的 byte，最低一位表示此域是否分词(tokenized)，倒数第二位表示此域是保存字符串数据还是二进制数据，倒数第 三位表示此域是否被压缩，再接下来就是存储域的值。

域索引文件（fdx）：

*   由域数据文件格式我们知道，每篇文档包􏰁的域的个数，每个存储域的值都是不一样的，因而域数据文件中 segment size 篇文档,每篇文档占用的大小也是不一样的, 那么如何在 fdt 中辨􏰀每一篇文档的起始地址和终止地址呢，如何能够更快的找到 第 n 篇文档的存储域的信息呢?就是要借助域索引文件。
*   域索引文件也总共有 segment size 个项，每篇文档都有一个项，每一项都是一个 long,大小固定，每一项都是对应的文档在 fdt 文件中的起始地址的偏移量,这样 如果我们想找到第 n 篇文档的存储域的信息，只要在 fdx 中找到第 n 项，然后按照 取出的 long 作为偏移量，就可以在 fdt 文件中找到对应的存储域的信息。

##### 4.1.4 词向量(Term Vector)的数据信息(.tvx,.tvd,.tvf)

![](/img/notes/search/lucenePrincipleAndCodeAnalysis/term_vector.png) 

词向量信息是从索引(index)到文档(document)到域(field)到词(term)的正向信息，有了词向量信息，我们就可以得到一篇文档包含􏰁哪些词的信息。

词向量索引文件(tvx)

*   一个段(segment)包􏰀 N 篇文档，此文件就有 N 项，每一项代表一篇文档。
*   每一项包􏰀两部分信息：第一部分是词向量文档文件(tvd)中此文档的偏移量，第二部分是词向量域文件(tvf)中此文档的第一个域的偏移量。

词向量文档文件(tvd)

*   一个段(segment)包􏰀 N 篇文档,此文件就有 N 项，每一项包􏰀了此文档的所有的域的信息。
*   每一项首先是此文档包􏰀的域的个数 NumFields，然后是一个 NumFields 大小的数组，数组的每一项是域号。然后是一个(NumFields - 1)大小的数组，由前面我们知 道，每篇文档的第一个域在 tvf 中的偏移量在 tvx 文件中保存，而其他(NumFields - 1) 个域在 tvf 中的偏移量就是第一个域的偏移量加上这(NumFields - 1)个数组的每一 项的值。

词向量域文件（tvf）

*   此文件包􏰀了此段中的所有的域，并不对文档做区分，到底第几个域到第几个域是属于那篇文档,是由 tvx 中的第一个域的偏移量以及 tvd 中的(NumFields - 1)个域的偏移量来决定的。
*   对于每一个域，首先是此域包􏰀的词的个数 NumTerms,然后是一个 8 位的 byte，最后一位是指定是否保存位置信息,倒数第二位是指定是否保存偏移量信息。然后 是 NumTerms 个项的数组,每一项代表一个词(Term)，对于每一个词,由词的文本 TermText，词频 TermFreq(也即此词在此文档中出现的次数)，词的位置信息，词的偏移量信息。

#### 4.2 反向信息

反向信息是索引文件的核心，也即反向索引。

反向索引包括两部分，左面是词典(Term Dictionary)，右面是倒排表(Posting List)。

在 Lucene 中，这两部分是分文件存储的,词典是存储在 tii,tis 中的，倒排表又包括两部分，一部分是文档号及词频，保存在 frq 中，一部分是词的位置信息，保存在 prx 中。

Term Dictionary(til, tis)：

*   Frequencies(.frq)
*   Positions(.prx)

##### 4.2.1 词典(tis)及词典索引(tii)信息

![](/img/notes/search/lucenePrincipleAndCodeAnalysis/tis_tii.png) 

在词典中，所有的词是按照字典顺序排序的。

词典文件（tis）：

*   TermCount：词典中包􏰀的总的词数
*   IndexInterval：为了加快对词的查找速度,也应用类似跳跃表的结构，假设IndexInterval 为 4，则在词典索引(tii)文件中保存第 4 个,第 8 个，第 12 个词，这样可以加快在词典文件中查找词的速度。
*   SkipInterval：倒排表无论是文档号及词频，还是位置信息，都是以跳跃表的结构存在的，SkipInterval 是跳跃的步数。
*   MaxSkipLevels：跳跃表是多层的，这个值指的是跳跃表的最大层数。
*   TermCount 个项的数组，每一项代表一个词，对于每一个词,以前缀后缀规则存放词的文本信息(PrefixLength + Suffix)，词属于的域的域号(FieldNum)，有多少篇文档 包􏰀此词(DocFreq)，此词的倒排表在 frq,prx 中的偏移量(FreqDelta, ProxDelta)，此词的倒排表的跳跃表在 frq 中的偏移量(SkipDelta)，这里之所以用 Delta，是应用差值规则。

词典索引文件（tii）：

*   词典索引文件是为了加快对词典文件中词的查找速度，保存每隔 IndexInterval 个词。
*   词典索引文件是会被全部加载到内存中去的。
*   IndexTermCount = TermCount / IndexInterval:词典索引文件中包􏰀的词数。
*   IndexInterval 同词典文件中的 IndexInterval。
*   SkipInterval 同词典文件中的 SkipInterval。
*   MaxSkipLevels 同词典文件中的 MaxSkipLevels。
*   IndexTermCount 个项的数组，每一项代表一个词,每一项包括两部分，第一部分是词本身(TermInfo)，第二部分是在词典文件中的偏移量(IndexDelta)。假设 IndexInterval 为 4，此数组中保存第 4 个，第 8 个，第 12 个词......

##### 4.2.2 文档号及词频（frq）信息

![](/img/notes/search/lucenePrincipleAndCodeAnalysis/frq.png) 

文档号及词频文件里面保存的是倒排表，是以跳跃表形式存在的。

*   此文件包􏰀 TermCount 个项，每一个词都有一项，因为每一个词都有自己的倒排表。
*   对于每一个词的倒排表都包括两部分，一部分是倒排表本身，也即一个数组的文档号及词频，另一部分是跳跃表，为了更快的访问和定位倒排表中文档号及词频的位置。

##### 4.2.3 词位置（prx）信息

![](/img/notes/search/lucenePrincipleAndCodeAnalysis/prx.png) 

词位置信息也是倒排表，也是以跳跃表形式存在的。

*   此文件包􏰀 TermCount 个项,每一个词都有一项，因为每一个词都有自己的词位置倒排表。
*   对于每一个词的都有一个 DocFreq 大小的数组，每项代表一篇文档，记录此文档中此词出现的位置。这个文档数组也是和 frq 文件中的跳跃表有关系的
*   对于每一篇文档，可能包􏰀一个词多次，因而有一个 Freq 大小的数组，每一项代表此词在此文档中出现一次，则有一个位置信息。
*   每一个位置信息包􏰀：PositionDelta(采用差值规则)，还可以保存 payload，应用或然跟随规则。

#### 4.3 其他信息

##### 4.3.1 标准化因子文件（nrm）

标准化因子（Normalization Factor）在索引过程总的计算如下：

![](/img/notes/search/lucenePrincipleAndCodeAnalysis/nrm.png)

它包括三个参数：

*   Document boost：此值越大，说明此文档越重要。
*   Field boost：此域越大，说明此域越重要。
*   lengthNorm(field) = (1.0 / Math.sqrt(numTerms))：一个域中包含的Term总数越多，也即文档越长，此值越小，文档越短，此值越大。

一个词(Term)出现在不同的文档或不同的域中，标准化因子不同。 比如有两个文档，每个文档有两个域，如果不考虑文档长度，就有四种排列组合，在重要文档的重要域中，在重要文档的非重要域中，在非重要文档的重要域中，在非重要文档的非重要域中，四种组合，每种有不同的标准化因子。

在 Lucene 中，标准化因子共保存了(文档数目乘以域数目)个，格式如下：

![](/img/notes/search/lucenePrincipleAndCodeAnalysis/nrm_format.png) 

标准化因子文件(Normalization Factor File: nrm)：

*   NormsHeader：字符串“NRM”外加 Version，依 Lucene 的版本的不同而不同。
*   接着是一个数组，大小为 NumFields，每个 Field 一项，每一项为一个 Norms。
*   Norms 也是一个数组，大小为 SegSize，即此段中文档的数量，每一项为一个 Byte，表示一个浮点数，其中 0~2 为尾数，3~8 为指数。

##### 4.3.2 删除文档文件（del）

![](/img/notes/search/lucenePrincipleAndCodeAnalysis/del.png) 

### 五、总体结构

![](/img/notes/search/lucenePrincipleAndCodeAnalysis/index_structure.png)

Lucene 索引文件的整体结构：

*  属于整个索引(Index)的 segment.gen,segment_N，其保存的是段(segment)的元数据信息，然后分多个 segment 保存数据信息，同一个 segment 有相同的前缀文件名。
*  对于每一个段，包􏰀域信息,词信息,以及其他信息(标准化因子,删除文档)
*  域信息也包括域的元数据信息，在 fnm 中，域的数据信息，在 fdx,fdt 中。
*  词信息是反向信息，包括词典(tis, tii)，文档号及词频倒排表(frq)，词位置倒排表(prx)。

## 第四章：Lucene索引过程分析

对于 Lucene 的索引过程，除了将词(Term)写入倒排表并最终写入 Lucene 的索引文件外，还包括分词(Analyzer)和合并段(merge segments)的过程

### 一、索引过程体系结构

![](/img/notes/search/lucenePrincipleAndCodeAnalysis/index_process_architecture.png)

索引过程，就是图中所示的索引链的过程。

*   基本索引链
*   线程索引链
*   域索引链

### 二、详细索引过程

#### 1、创建IndexWriter对象

IndexWriter对象主要包含以下几方面的信息：

*   用于索引文档
    *   Directory directory; 指向索引文件夹
    *   Analyzer analyzer; 分词器
    *   Similarity similarity = Similarity.getDefault(); 影响打分的标准化因子(normalization factor)部分，对文档的打分分两个部分，一部分是索引阶段计算的，与查询语句无关，一部分是搜索阶段计算的，与查询语句相关。
    *   SegmentInfos segmentInfos = new SegmentInfos(); 保存段信息，和segments_N 中的信息几乎一一对应。
    *   IndexFileDeleter deleter; 此对象不是用来删除文档的，而是用来管理索引文件的。
    *   Lock writeLock; 每一个索引文件夹只能打开一个 IndexWriter，所以需要锁。
    *   Set<SegmentInfo> segmentsToOptimize = new HashSet<SegmentInfo>(); 保存正在最优化(optimize)的段信息。当调用 optimize 的时候，当前所有的段信息加入此 Set，此后新生成的段并不参与此次最优化。
*   用于合并段，在合并段的文章中将详细描述
*   为保持索引完整性，一致性和事务性
    *   SegmentInfos rollbackSegmentInfos; 当 IndexWriter 对索引进行了添加，删除文档操作后，可以调用 commit 将修改提交到文件中去，也可以调用 rollback 取消从上次 commit 到此时的修改。
    *   此段信息主要用于将其他的索引文件夹合并到此索引文件夹的时候，为防止合并到一半出错可回滚所保存的原来的段信息。
*   一些配置
    *   long writeLockTimeout; 获得锁的时间超时。当超时的时候，说明此索引文件夹已经被另一个 IndexWriter 打开了。
    *   int termIndexInterval; 同 tii 和 tis 文件中的 indexInterval。

有关IndexFileDeleter：

*   其不是用来删除文档的,而是用来管理索引文件的。
*   在对文档的添加，删除，对段的合并的处理过程中，会生成很多新的文件，并需要删除老的文件，因而需要管理。
*   然而要被删除的文件又可能在被用，因而要保存一个引用计数，仅仅当引用计数为零的时候，才执行删除。

#### 2、创建文档Document对象，并加入域（Field）

Document对象主要包括以下部分：

*   此文档的 boost，默认为 1，大于一说明比一般的文档更加重要，小于一说明更不重要。
*   一个 ArrayList 保存此文档所有的域
*   每一个域包括域名,域值,和一些标志位，和 fnm,fdx,fdt 中的描述相对应。

#### 3、将文档加入IndexWriter

IndexWriter 继而调用 DocumentsWriter.addDocument，其又调用 DocumentsWriter.updateDocument。

#### 4、将文档加入DocumentsWriter

DocumentsWriter 对象主要包􏰀以下几部分：

*   用于写索引文件
    *   IndexWriter writer;
    *   Directory directory;
    *   Similarity similarity:分词器
    *   String segment：当前的段名，每当 flush 的时候，将索引写入以此为名称的段。
    *   String docStoreSegment：存储域所要写入的目标段。
    *   int docStoreOffset：存储域在目标段中的偏移量。
    *   int nextDocID：下一篇添加到此索引的文档 ID 号，对于同一个索引文件夹，此变量唯一，且同步访问。
    *   DocConsumer consumer; 这是整个索引过程的核心，是 IndexChain 整个索引链的源头。
*   删除文档
*   缓存管理
*   多线程并发索引
*   一些标志位

##### 4.1、得到当前线程对应的文档集处理对象(DocumentsWriterThreadState)

为了支持多线程索引，不使 IndexWriter 成为瓶颈，对于每一个线程都有一个相应的文档集处理对象(DocumentsWriterThreadState)，这样对文档的索引过程可以多线程并行进行, 从而增加索引的速度。

DocumentsWriterThreadState DocumentsWriter.getThreadState(Document doc, Term delTerm)包 􏰀如下几个过程：

*   根据当前线程对象，从 HashMap 中查找相应的 DocumentsWriterThreadState 对象，如果没找到，则生成一个新对象，并添加到 HashMap 中
*   如果此线程对象正在用于处理上一篇文档，则等待，直到此线程的上一篇文档处理完。
*   如果 IndexWriter 刚刚 commit 过，则新添加的文档要加入到新的段中(segment)，则首先要生成新的段名。
*   将此线程的文档处理对象设为忙碌:state.isIdle = false;

##### 4.2、用得到的文档集处理对象（DocumentsWriterThreadState）处理文档

每一个文档集处理对象 DocumentsWriterThreadState 都有一个文档及域处理对象 DocFieldProcessorPerThread，它的成员函数 processDocument()被调用来对文档及域进行处理。

DocumentsWriter.DocWriter DocFieldProcessorPerThread.processDocument()包􏰀以下几个过程：

###### 4.2.1、开始处理当前文档

在此版的 Lucene 中，几乎所有的 XXXPerThread 的类，都有 startDocument 和 finishDocument两个函数，因为对同一个线程，这些对象都是复用的，而非对每一篇新来的文档都创建一套，这样也提高了效率,也牵扯到数据的清理问题。一般在 startDocument 函数中,清理处理上 篇文档遗留的数据，在 finishDocument 中，收集本次处理的结果数据，并返回，一直返回到 DocumentsWriter.updateDocument(Document, Analyzer, Term) 然后根据条件判断是否将数据刷新到硬盘上。