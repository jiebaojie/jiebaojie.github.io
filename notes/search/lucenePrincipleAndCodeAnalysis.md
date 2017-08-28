---
layout: post
notes: true
subtitle: Lucene原理与代码分析完整版
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

###### 4.2.2、逐个处理文档的每一个域

*   4.2.2.1、首先：对于每一个域，按照域名，在 fieldHash 中查找域处理对象 DocFieldProcessorPerField
*   4.2.2.2、然后：对 fields 数组进行排序，是域按照名称排序。quickSort(fields, 0, fieldCount-1);
*   4.2.3.3、最后：按照排序号的顺序，对域逐个处理，此处处理的仅仅是索引域

###### 4.2.3、结束处理当前文档

final DocumentsWriter.DocWriter one = fieldsWriter(StoredFieldsWriterPerThread).finishDocument();

存储域返回结果：一个写成了二进制的存储域缓存。

##### 4.3、用 DocumentsWriter.finishDocument 结束本次文档添加

#### 5、DocumentsWriter 对 CharBlockPool,ByteBlockPool，IntBlockPool 的缓存管理

*   在索引的过程中,DocumentsWriter 将词信息(term)存储在 CharBlockPool 中，将文档号 (doc ID),词频(freq)和位置(prox)信息存储在 ByteBlockPool 中。
*   在 ByteBlockPool 中，缓存是分块(slice)分配的，块(slice)是分层次的，层次越高，此层的块越大，每一层的块大小事相同的。
    *   nextLevelArray 表示的是当前层的下一层是第几层
    *   levelSizeArray 表示每一层的块大小

#### 6、关闭IndexWriter对象

将索引写入磁盘包括以下几个过程:

*   得到要写入的段名:String segment = docWriter.getSegment();
*   DocumentsWriter 将缓存的信息写入段:docWriter.flush(flushDocStores);
*   生 成 新 的 段 信 息 对 象 : newSegment = new SegmentInfo(segment, flushedDocCount, directory, false, true, docStoreOffset, docStoreSegment, docStoreIsCompoundFile, docWriter.hasProx());
*   准备删除文档:docWriter.pushDeletes();
*   生成 cfs 段:docWriter.createCompoundFile(segment);
*   删除文档：applyDeletes();

##### 6.1、得到要写入的段名

存储域和词向量可以和索引域存储在不同的段中。

##### 6.2、将缓存的内容写入段

此过程又包􏰀以下两个阶段：

###### 6.2.1、 按照基本索引链关闭存储域和词向量信息

其主要是根据基本索引链结构，关闭存储域和词向量信息

*   词向量的关闭：TermVectorsTermsWriter.closeDocStore(SegmentWriteState)
*   存储域的关闭：StoredFieldsWriter.closeDocStore(SegmentWriteState)

###### 6.2.2、按照基本索引链的结构将索引结果写入段

此过程也是按照基本索引链来的：

*   6.2.2.1、写入存储域
*   6.2.2.2、写入索引域
    *   6.2.2.2.1、写入倒排表及词向量信息
        *   6.2.2.2.1.1、写入倒排表信息
        *   6.2.2.2.1.2、写入词向量信息
    *   6.2.2.2.2、写入标准化因子
*   6.2.2.3、写入域元数据

##### 6.3、生成新的段信息对象

##### 6.4、准备删除文档

此处将 deletesInRAM 全部加到 deletesFlushed 中，并把 deletesInRAM 清空。

##### 6.5、生成cfs段

##### 6.6、删除文档

Lucene删除文档可以用reader，也可以用writer，但是归根结底还是用reader来删除的。

reader的删除有以下三种方式：

*   按照词删除，删除所有包􏰀此词的文档。
*   按照文档号删除。
*   按照查询对象删除，删除所有满足此查询的文档。

但是这三种方式归根结底还是按照文档号删除，也就是写.del文件的过程。

## 第五章：Lucene 段合并(merge)过程分析

### 一、段合并过程总论

IndexWriter 中与段合并有关的成员变量有：

*   HashSet<SegmentInfo> mergingSegments = new HashSet<SegmentInfo>(); //保存正在合并的段，以防止合并期间再次选中被合并。
*   MergePolicy mergePolicy = new LogByteSizeMergePolicy(this); //合并策略，也即选取哪些段来进行合并。
*   MergeScheduler mergeScheduler = new ConcurrentMergeScheduler(); //段合并器，背后有一个线程负责合并。
*   LinkedList<MergePolicy.OneMerge> pendingMerges = new LinkedList<MergePolicy.OneMerge>(); //等待被合并的任务
*   Set<MergePolicy.OneMerge> runningMerges = new HashSet<MergePolicy.OneMerge>(); // 正在被合并的任务

和段合并有关的一些参数有：

*   mergeFactor：当大小几乎相当的段的数量达到此值的时候，开始合并。
*   minMergeSize：所有大小小于此值的段，都被认为是大小几乎相当，一同参与合并。
*   maxMergeSize：当一个段的大小大于此值的时候，就不再参与合并。
*   maxMergeDocs：当一个段包􏰀的文档数大于此值的时候，就不再参与合并。

段合并一般发生在添加完一篇文档的时候，当一篇文档添加完后，发现内存已经达到用户设 定的ramBufferSize，则写入文件系统，形成一个新的段。新段的加入可能造成差不多大小的段的个数达到 mergeFactor，从而开始了合并的过程。

合并过程最重要的两部分：

*   一个是选择哪些段应该参与合并，这一步由MergePolicy来决定。
*   一个是将选择出的段合并成新段的过程，这一步由MergeScheduler来执行。段的合并也主要包括：
    *   对正向信息的合并，如存储域，词向量，标准化因子等。
    *   对反向信息的合并，如词典，倒排表。

#### 1.1、合并策略对段的选择

#### 1.2、反向信息的合并

反向信息的合并包括两部分：

*   对字典的合并，词典中的 Term 是按照字典顺序排序的，需要对词典中的 Term 进行重新排序
*   对于相同的 Term，对包􏰁此 Term 的文档号列表进行合并，需要对文档号重新编号

### 二、段合并的详细过程

#### 2.1、将缓存写入新的段

当缓存写入硬盘，形成了新的段后，就有可能触发一次段合并，所以调用 maybeMerge()

#### 2.2、选择合并段，生成合并任务

##### 2.2.1、用合并策略选择合并段

##### 2.2.2、注册段合并任务

#### 2.3、段合并器进行段合并

##### 2.3.1、合并存储域

合并存储域主要包􏰁两部分：一部分是合并 fnm 信息，也即域元数据信息，一部分是合并 fdt,fdx 信息，也即域数据信息。

##### 2.3.2、合并标准化因子

合并标准化因子的过程比较简单，基本就是对每一个域，用指向合并段的 reader 读出标准 化因子，然后再写入新生成的段。

##### 2.3.3、合并词向量

##### 2.3.4、合并词典和倒排表

反向信息的合并包括两部分：

*   对字典的合并，需要对词典中的 Term 进行重新排序
*   对于相同的 Term，对包􏰁此 Term 的文档号列表进行合并，需要对文档号重新编号。

## 第六章：Lucene打分公式的数学推导

Lucene的搜索过程，很重要的一个步骤就是逐步的计算各部分的分数。

Lucene的打分公式：

![](/img/notes/search/lucenePrincipleAndCodeAnalysis/score.png)

每部分的意义：

*	t：Term，这里的 Term 是指包含域信息的 Term，也即 title:hello 和 content:hello 是不同的 Term
*	coord(q,d)：一次搜索可能包含多个搜索词，而一篇文档中也可能包括多个搜索词，此项表示，当一篇文档中包含的搜索词越多，则此文档则打分越高。
*	queryNorm(q)：计算每个查询条目的方差和，此值并不影响排序，而仅仅使得不同的 query 之间的分数可以比较。其公式如下：

![](/img/notes/search/lucenePrincipleAndCodeAnalysis/query_norm.png)

*	tf(t in d)：Term t 在文档 d 中出现的词频
*	idf(t)：逆向文档频率，idf(t)= 1 +log(文档总数/(包含t的文档数+1))
*	norm(t, d)：标准化因子，它包括三个参数：
	*	Document boost：此值越大，说明此文档越重要。
	*	Field boost：此域越大，说明此域越重要。 
	*	lengthNorm(field) = (1.0 / Math.sqrt(numTerms))：一个域中包含的 Term 总数越多，也即文档越长，此值越小，文档越短，此值越大。 

![](/img/notes/search/lucenePrincipleAndCodeAnalysis/norm.png)

![](/img/notes/search/lucenePrincipleAndCodeAnalysis/length_norm.png)

*	各类 Boost 值
	*	t.getBoost()：查询语句中每个词的权重，可以在查询中设定某个词更加重要，common^4 hello
	*	d.getBoost()：文档权重，在索引阶段写入 nrm 文件，表明某些文档比其他文档更重要。
	*	f.getBoost()：域的权重，在索引阶段写入 nrm 文件，表明某些域比其他的域更重要。

，如何调整上面的各部分，以影响文档的打分，请参考[有关Lucene的问题(4):影响Lucene对文档打分的四种方式](http://www.cnblogs.com/forfuture1978/archive/2010/02/08/1666137.html)一文。

### 推导

首先，将以上各部分代入 score(q, d)公式，将得到一个非常复杂的公式，让我们忽略所有的 boost，因为这些属于人为的调整，也省略 coord，这和公式所要表达的原理无关。得到下面的公式：

![](/img/notes/search/lucenePrincipleAndCodeAnalysis/without_boost.png)	

我们把文档看作一系列词(Term)，每一个词(Term)都有一个权重(Term weight)，不同的词(Term)根据自己在文档中的权重来影响文档相关性的打分计算。

于是我们把所有此文档中词(term)的权重(term weight) 看作一个向量。

Document = {term1, term2, …… ,term N}

Document Vector = {weight1, weight2, …… ,weight N}

同样我们把查询语句看作一个简单的文档，也用向量来表示。

Query = {term1, term 2, …… , term N}

Query Vector = {weight1, weight2, …… , weight N}

我们把所有搜索出的文档向量及查询向量放到一个N维空间中，每个词(term)是一维。

![](/img/notes/search/lucenePrincipleAndCodeAnalysis/vector_cos.png)

我们认为两个向量之间的夹角越小，相关性越大。

所以我们计算夹角的余弦值作为相关性的打分，夹角越小，余弦值越大，打分越高，相关性越大。

余弦公式如下：

![](/img/notes/search/lucenePrincipleAndCodeAnalysis/score_q_d.png)	

下面我们假设：

查询向量为Vq = <w(t1, q), w(t2, q), ……, w(tn, q)>

文档向量为Vd = <w(t1, d), w(t2, d), ……, w(tn, d)>

向量空间维数为n，是查询语句和文档的并集的长度，当某个Term不在查询语句中出现的时候，w(t, q)为零，当某个Term不在文档中出现的时候，w(t, d)为零。

w代表weight，计算公式一般为tf*idf。

我们首先计算余弦公式的分子部分，也即两个向量的点积：

Vq*Vd = w(t1, q)*w(t1, d) + w(t2, q)*w(t2, d) + …… + w(tn ,q)*w(tn, d)

把w的公式代入，则为

Vq*Vd = tf(t1, q)*idf(t1, q)*tf(t1, d)*idf(t1, d) + tf(t2, q)*idf(t2, q)*tf(t2, d)*idf(t2, d) + …… + tf(tn ,q)*idf(tn, q)*tf(tn, d)*idf(tn, d)

在这里有三点需要指出：

*	由于是点积，则此处的t1, t2, ……, tn只有查询语句和文档的并集有非零值，只在查询语句出现的或只在文档中出现的Term的项的值为零。
*	在查询的时候，很少有人会在查询语句中输入同样的词，因而可以假设tf(t, q)都为1
*	idf是指逆向文档频率，其中也包括查询语句这篇小文档，因而idf(t, q)和idf(t, d)其实是一样的，是索引中的文档总数加一，当索引中的文档总数足够大的时候，查询语句这篇小文档可以忽略，因而可以假设idf(t, q) = idf(t, d) = idf(t)

基于上述三点，点积公式为：

Vq*Vd = tf(t1, d) * idf(t1) * idf(t1) + tf(t2, d) * idf(t2) * idf(t2) + …… + tf(tn, d) * idf(tn) * idf(tn)

所以余弦公式变为：

![](/img/notes/search/lucenePrincipleAndCodeAnalysis/score_q_d_cos.png)

下面要推导的就是查询语句的长度了。

由上面的讨论，查询语句中tf都为1，idf都忽略查询语句这篇小文档，得到如下公式

![](/img/notes/search/lucenePrincipleAndCodeAnalysis/vq_length.png)

所以余弦公式变为：

![](/img/notes/search/lucenePrincipleAndCodeAnalysis/score_qd_vq.png)

下面推导的就是文档的长度了，本来文档长度的公式应该如下:

![](/img/notes/search/lucenePrincipleAndCodeAnalysis/vd_length.png)

这里需要讨论的是，为什么在打分过程中，需要除以文档的长度呢？

因为在索引中，不同的文档长度不一样，很显然，对于任意一个term，在长的文档中的tf要大的多，因而分数也越高，这样对小的文档不公平，举一个极端的例子，在一篇1000万个词的鸿篇巨著中，"lucene"这个词出现了11次，而在一篇12个词的短小文档中，"lucene"这个词出现了10次，如果不考虑长度在内，当然鸿篇巨著应该分数更高，然而显然这篇小文档才是真正关注"lucene"的。

然而如果按照标准的余弦计算公式，完全消除文档长度的影响，则又对长文档不公平(毕竟它是包含了更多的信息)，偏向于首先返回短小的文档的，这样在实际应用中使得搜索结果很难看。

所以在Lucene中，Similarity的lengthNorm接口是开放出来，用户可以根据自己应用的需要，改写lengthNorm的计算公式。比如我想做一个经济学论文的搜索系统，经过一定时间的调研，发现大多数的经济学论文的长度在8000到10000词，因而lengthNorm的公式应该是一个倒抛物线型的，8000到 10000词的论文分数最高，更短或更长的分数都应该偏低，方能够返回给用户最好的数据。

在默认状况下，Lucene采用DefaultSimilarity，认为在计算文档的向量长度的时候，每个Term的权重就不再考虑在内了，而是全部为一。

而从Term的定义我们可以知道，Term是包含域信息的，也即title:hello和content:hello是不同的Term，也即一个Term只可能在文档中的一个域中出现。

所以文档长度的公式为：

![](/img/notes/search/lucenePrincipleAndCodeAnalysis/vd_normal.png)

代入余弦公式：

![](/img/notes/search/lucenePrincipleAndCodeAnalysis/score_q_d_final.png)

再加上各种boost和coord，则可得出Lucene的打分计算公式。

## 第七章：Lucene搜索过程解析

### 一、Lucene搜索过程总论

搜索的过程总的来说就是将词典及倒排表信息从索引中读出来，根据用户输入的查询语句合并倒排表，得到结果文档集并对文档进行打分的过程。

![](/img/notes/search/lucenePrincipleAndCodeAnalysis/search_score.png)

包括以下几个过程：

1.  IndexReader 打开索引文件，读取并打开指向索引文件的流。
2.  用户输入查询语句
3.  将查询语句转换为查询对象 Query 对象树
4.  构造 Weight 对象树，用于计算词的权重 Term Weight，也即计算打分公式中与仅与搜索语句相关与文档无关的部分(红色部分)。
5.  构造 Scorer 对象树,用于计算打分(TermScorer.score())。
6.  在构造 Scorer 对象树的过程中，其叶子节点的 TermScorer 会将词典和倒排表从索引中读出来。
7.  构造 SumScorer 对象树，其是为了方便合并倒排表对 Scorer 对象树的从新组织，它的叶 子节点仍为 TermScorer，包􏰀词典和倒排表。
8.  将收集的结果集合及打分返回给用户。

### 二、Lucene搜索详细过程

#### 2.1、打开IndexReader指向索引文件夹

##### 2.1.1、找到最新的segment_N文件

##### 2.1.2、通过 segment_N 文件中保存的各个段的信息打开各个段

#### 2.2、打开IndexSearcher

#### 2.3、QueryParser解析查询语句生成查询对象

#### 2.4、搜索查询对象

##### 2.4.1、创建 Weight 对象树，计算 Term Weight

###### 2.4.1.1、重写Query对象树

###### 2.4.1.2、创建Weight对象树

###### 2.4.1.3、计算 Term Weight 分数

##### 2.4.2、创建 Scorer 及 SumScorer 对象树

##### 2.4.3、进行倒排表合并

###### 2.4.3.1、交集 ConjunctionScorer(+A +B)

###### 2.4.3.2、并集 DisjunctionSumScorer(A OR B)

###### 2.4.3.3、差集 ReqExclScorer(+A -B)

###### 2.4.3.4、ReqOptSumScorer(+A B)

###### 2.4.3.4、有关 BooleanScorer 及 scoresDocsOutOfOrder

##### 2.4.4、收集文档结果集合及计算打分

###### 2.4.4.1、创建结果文档收集器

###### 2.4.4.2、收集文档号

###### 2.4.4.3、打分计算

###### 2.4.4.4、返回打分最高的 N 篇文档

##### 2.4.5、 Lucene如何在搜索阶段读取索引信息

###### 2.4.5.1、读取词典信息

###### 2.4.5.2、读取倒排表信息

## 第八章：Lucene的查询语法，JavaCC及QueryParser

### 一、Lucene的查询语法

[Lucene所支持的查询语法](http://lucene.apache.org/java/3_0_1/queryparsersyntax.html)

#### （1）语法关键字

+ - && || ! ( ) { } [ ] ^ " ~ * ? : \

如果所要查询的查询词中本身包􏰀关键字，则需要用\进行转义

#### （2）查询词（Term）

Lucene 支持两种查询词，一种是单一查询词，如"hello"，一种是词组(phrase)，如"hello world"。

#### （3）查询域（Field）

在查询语句中，可以指定从哪个域中寻找查询词，如果不指定，则从默认域中查找。

查询域和查询词之间用:分隔，如 title:"Do it right"。

:仅对紧跟其后的查询词起作用，如果 title:Do it right，则仅表示在 title 中查询 Do，而 it right 要在默认域中查询。

#### （4）通配符查询（Wildcard）

支持两种通配符：?表示一个字符，*表示多个字符。

通配符可以出现在查询词的中间或者􏰁尾，但决不能出现在开始。

#### （5）模糊查询（Fuzzy）

模糊查询的算法是基于 Levenshtein Distance，也即当两个词的差􏰂小于某个比例的时候，就算匹配，如 roam~0.8，即表示差􏰂小于 0.2，相似度大于 0.8 才算匹配。

#### （6）临近查询（Proximity）

#### （7） 区间查询（Range）

区间查询包􏰀两种，一种是包􏰀边界，用[A TO B]指定，一种是不包􏰀边界，用{A TO B} 指定。

#### （8）增加一个查询的权重（Boost）

可以在查询词后面加^N 来设定此查询词的权重，默认是 1，如果 N 大于 1，则说明此查询词更重要，如果 N 小于 1，则说明此查询词更不重要。

如 jakarta^4 apache，"jakarta apache"^4 "Apache Lucene"

#### （9）布尔操作符

布尔操作符包括连接符，如 AND,OR，和修饰符，如 NOT,+,-。

默认状态下，空格被认为是 OR 的关系。

QueryParser.setDefaultOperator(Operator.AND)设置为空格为 AND。

#### （10）组合

可以用括号，将查询语句进行组合，从而设定优先级。

如(jakarta OR apache) AND website

Lucene 的查询语法是由 QueryParser 来进行解析，从而生成查询对象的。

QueryParser 是通过 JavaCC 来生成词法分析器和语法分析器的。

### 二、JavaCC介绍

#### 2.1、第一个实例——正整数相加

#### 2.2、扩展语法分析器

#### 2.3、第二个实例：计算器

### 三、解析QueryParser.jj

#### 3.1、声明QueryParser类

#### 3.2、声明词法分析器

#### 3.3、声明语法分析器

## 第九章：Lucene的查询对象

### 1、BoostingQuery

### 2、CustomScoreQuery

### 3、MoreLikeThisQuery

### 4、MultiTermQuery

#### 4.1、TermRangeQuery

#### 4.2、NumericRangeQuery

### 5、SpanQuery

#### 5.1、SpanFirstQuery

#### 5.2、SpanNearQuery

#### 5.3、SpanNotQuery

#### 5.4、SpanOrQuery

#### 5.5、FieldMaskingSpanQuery

#### 5.6、PayloadTermQuery及PayloadNearQuery

### 6、FilteredQuery

#### 6.1、TermsFilter

#### 6.2、BooleanFilter

#### 6.3、DuplicateFilter

#### 6.4、FieldCacheRangeFilter<T>及FieldCacheTermsFilter

#### 6.5、MultiTermQueryWrapperFilter<Q>

#### 6.6、QueryWrapperFilter

#### 6.7、SpanFilter

##### 6.7.1、SpanQueryFilter

##### 6.7.2、CachingSpanFilter

## 第十章：Lucene的分词器Analyzer

### 1、抽象类Analyzer

其主要包􏰀两个接口，用于生成 TokenStream：

*   TokenStream tokenStream(String fieldName, Reader reader);
*   TokenStream reusableTokenStream(String fieldName, Reader reader) ;

TokenStream是一个由分词后的 Token 结果组成的流，能够不断的得到下一个分成的 Token。

为了提高性能，使得在同一个线程中无需再生成新的 TokenStream 对象，老的可以被重用，所以有 reusableTokenStream 一说。

Analyzer 中有 CloseableThreadLocal<Object> tokenStreams = new CloseableThreadLocal<Object>(); 成员变量，保存当前线程原来创建过的 TokenStream，可用函数 setPreviousTokenStream 设定，用函数 getPreviousTokenStream 得到。

在 reusableTokenStream 函数中，往往用 getPreviousTokenStream 得到老的 TokenStream 对象，然后将 TokenStream 对象 reset 以下，从而可以从新开始得到 Token 流。

### 2、TokenStream抽象类

TokenStream主要包含以下几个方法：

*   boolean incrementToken()用于得到下一个 Token。
*   public void reset() 使得此 TokenStrean 可以重新开始返回各个分词。

### 3、几个具体的TokenStream

在索引的时候，添加域的时候，可以指定 Analyzer，使其生成 TokenStream，也可以直接指定 TokenStream

#### 3.1、NumericTokenStream

#### 3.2、SingleTokenTokenStream

### 4、Tokenizer也是一种TokenStream

#### 4.1、CharTokenizer

CharTokenizer 是一个抽象类，用于对字符串进行分词。

#### 4.2、ChineseTokenizer

#### 4.3、KeywordTokenizer

KeywordTokenizer 是将整个字符作为一个 Token 返回的。

#### 4.4、CJKTokenizer

#### 4.5、SentenceTokenizer

其是按照如下的标点来拆分句子。

### 5、TokenFilter也是一种TokenStream

来对 Tokenizer 后的 Token 作过滤，其使用的是装饰者模式。

#### 5.1、ChineseFilter

#### 5.2、LengthFilter

#### 5.3、LowerCaseFilter

#### 5.4、NumbericPayloadTokenFilter

#### 5.5、PorterStemFilter

#### 5.6、ReverseStringFilter

#### 5.7、SnowballFilter

其包􏰀成员变量 SnowballProgram stemmer，其是一个抽象类，其子类有 EnglishStemmer 和 PorterStemmer 等。

#### 5.8、TeeSinkTokenFilter

TeeSinkTokenFilter 可以使得已经分好词的 Token 全部或者部分的被保存下来，用于生成另一个 TokenStream 可以保存在其他的域中。

### 6、不同的 Analyzer 就是组合不同的 Tokenizer 和 TokenFilter 得到最后的 TokenStream

#### 6.1、ChineseAnalyzer

按字分词，并过滤停词，标点，英文

举例："Thisyear, president Hu 科学发展观" 被分词为 "year","president","hu","科","学","发"," 展","观"

#### 6.2、CJKAnalyzer

每两个字组成一个词，并去除停词

举例："This year, president Hu 科学发展观" 被分词为"year","president","hu","科学","学发"," 发展","展观"。

#### 6.3、PorterStemAnalyzer

将转为小写的 token，利用 porter 算法进行 stemming

#### 6.4、SmartChineseAnalyzer

先分句子，句子中分词组，用porter算法进行stemming，去停词

#### 6.5、SnowballAnalyzer

使用标准的分词器，标准的过滤器，转换为小写，去停词，根据设定的stemmer进行stemming

### 7、Lucene的标准分词器

#### 7.1、StandardTokenizerImpl.jflex

和 QueryParser 类似，标准分词器也需要词法分析，在原来的版本中，也是用 javacc，当前的版本中，使用的是 jflex。

jflex 也是一个词法及语法分析器的生成器，它主要包括三部分，由%%分隔:

*   用户代码部分：多为 package 或者 import
*   选项及词法声明
*   语法规则声明

#### 7.2、StandardTokenizer

#### 7.3、StandardFilter

#### 7.4、StandardAnalyzer

### 8、不同的域使用不同的分词器

#### 8.1、PerFieldAnalyzerWrapper

有时候，我们想不同的域使用不同的分词器，则可以用 PerFieldAnalyzerWrapper 进行封装。

其有两个成员函数：

*   Analyzer defaultAnalyzer：即当域没有指定分词器的时候使用此分词器
*   Map<String,Analyzer> analyzerMap = new HashMap<String,Analyzer>()：一个从域名到分词器的映射，将根据域名使用相应的分词器。

# 第三篇：问题篇

