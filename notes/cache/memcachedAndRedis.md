---
layout: post
notes: true
subtitle: "【技术博客】memcached与redis实现的对比"
comments: false
author: "私募工场"
date: 2016-11-27 00:00:00

---


原文：[https://www.qcloud.com/community/article/129](https://www.qcloud.com/community/article/129)

*   目录
{:toc }

# 一、综述

memcached和redis就是将数据存储在内存中，按照key-value的方式查询，可以大幅度提高效率。所以一般它们都用做缓存服务器，缓存常用的数据，需要查询的时候，直接从它们那儿获取，减少查询数据库的次数，提高查询效率。

# 二、服务方式

memcached和redis自己本身就是网络服务器，用户进程通过与他们通过网络来传输数据，显然最简单和最常用的就是使用tcp连接了。另外，memcached和redis都支持udp协议。而且当用户进程和memcached和redis在同一机器时，还可以使用unix域套接字通信。

# 三、事件模型

redis的事件模型很简单，只有一个event loop，是简单的reactor实现。

而memcached是多线程的，使用master-worker的方式，主线程监听端口，建立连接，然后顺序分配给各个工作线程。每一个从线程都有一个event loop，它们服务不同的客户端。

多线程的优势就是可以充分发挥多核的优势，不过编写程序麻烦一点，memcached里面就有各种锁和条件变量来进行线程同步。

# 四、内存分配

memcached和redis的核心任务都是在内存中操作数据，内存管理自然是核心的内容。

*   memcached是有自己得内存池的，即预先分配一大块内存，然后接下来分配内存就从内存池中分配，这样可以减少内存分配的次数，提高效率，这也是大部分网络服务器的实现方式，只不过各个内存池的管理方式根据具体情况而不同。
*   redis没有自己得内存池，而是直接使用时分配，即什么时候需要什么时候分配，内存管理的事交给内核，自己只负责取和释放。不过redis支持使用tcmalloc来替换glibc的malloc，前者是google的产品，比glibc的malloc快。

redis既是单线程，又没有自己的内存池，是不是感觉实现的太简单了？那是因为它的重点都放在数据库模块了

由于redis没有自己的内存池，所以内存申请和释放的管理就简单很多，直接malloc和free即可，十分方便。而memcached是支持内存池的，所以内存申请是从内存池中获取，而free也是还给内存池，所以需要很多额外的管理操作，实现起来麻烦很多

# 五、数据库实现

## 1. memcached数据库实现

memcached只支持key-value，即只能一个key对于一个value。它的数据在内存中也是这样以key-value对的方式存储，它使用slab机制。

每一个key-value对都存储在一个item结构中，包含了相关的属性和key和value的值。

![](/img/notes/cache/memcachedAndRedis/memcached_item.jpg)

item是保存key-value对的，当item多的时候，怎么查找特定的item是个问题。所以memcached维护了一个hash表，它用于快速查找item。hash表使用开链法（与redis一样）解决键的冲突，每一个hash表的桶里面存储了一个链表，链表节点就是item的指针，如上图中的h_next就是指桶里面的链表的下一个节点。

memcached支持设置过期时间，即expire time，但是内部并不定期检查数据是否过期，而是客户进程使用该数据的时候，memcached会检查expire time，如果过期，直接返回错误。这样的优点是，不需要额外的cpu来进行expire time的检查，缺点是有可能过期数据很久不被使用，则一直没有被释放，占用内存。

memcached是多线程的，而且只维护了一个数据库，所以可能有多个客户进程操作同一个数据，这就有可能产生问题。比如，A已经把数据更改了，然后B也更改了改数据，那么A的操作就被覆盖了，而可能A不知道，A任务数据现在的状态时他改完后的那个值，这样就可能产生问题。为了解决这个问题，memcached使用了CAS协议，简单说就是item保存一个64位的unsigned int值，标记数据的版本，每更新一次（数据值有修改），版本号增加，然后每次对数据进行更改操作，需要比对客户进程传来的版本号和服务器这边item的版本号是否一致，一致则可进行更改操作，否则提示脏数据。

## 2. redis数据库实现

redis数据库的功能强大一些，因为不像memcached只支持保存字符串，redis支持string， list， set，sorted set，hash table 5种数据结构。

为了实现这些数据结构，redis定义了抽象的对象redis object，如下图。每一个对象有类型，一共5种：字符串，链表，集合，有序集合，哈希表。 同时，为了提高效率，redis为每种类型准备了多种实现方式，根据特定的场景来选择合适的实现方式，encoding就是表示对象的实现方式的。

然后还有记录了对象的lru，即上次被访问的时间，同时在redis 服务器中会记录一个当前的时间（近似值，因为这个时间只是每隔一定时间，服务器进行自动维护的时候才更新），它们两个只差就可以计算出对象多久没有被访问了。

然后redis object中还有引用计数，这是为了共享对象，然后确定对象的删除时间用的。

最后使用一个void*指针来指向对象的真正内容。正式由于使用了抽象redis object，使得数据库操作数据时方便很多，全部统一使用redis object对象即可，需要区分对象类型的时候，再根据type来判断。而且正式由于采用了这种面向对象的方法，让redis的代码看起来很像c++代码，其实全是用c写的。

    typedef struct redisObject {
        unsigned type:4;            // 对象的类型，包括 /* Object types */
        unsigned encoding:4;        // 底部为了节省空间，一种type的数据，可以采用不同的存储方式
        unsigned lru:REDIS_LRU_BITS;// lru time (relative to server.lruclock)
        int refcount;               // 引用计数
        void *ptr; 
    }

说到底redis还是一个key-value的数据库，不管它支持多少种数据结构，最终存储的还是以key-value的方式，只不过value可以是链表，set，sorted set，hash table等。

和memcached一样，所有的key都是string，而set，sorted set，hash table等具体存储的时候也用到了string。 而c没有现成的string，所以redis的首要任务就是实现一个string，取名叫sds（simple dynamic string），如下的代码， 非常简单的一个结构体，len存储改string的内存总长度，free表示还有多少字节没有使用，而buf存储具体的数据，显然len-free就是目前字符串的长度。

    struct sdshdr {
        int len;
        int free;
        char buf[];
    }