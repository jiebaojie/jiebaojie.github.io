---
layout: post
title: Java8中HashMap的优化
comments: true
author: "Bao Jie"
date: 2018-01-28 00:00:00
header-img: 
tags:
    - Java8
    - HashMap
---

# 1. Java8之前的HashMap

Java8 之前的HashMap，基于一个数组以及多个链表的实现，hash值冲突的时候，就将对应节点以链表的形式存储。在hashcode特别差的情况下，比如说所有的Key的hashcode都相同，这个链表可能会很长，那么put/get操作都可能要遍历这个链表。也就是说时间复杂度在最差情况下会退化到O(n)。

比如有一种攻击叫Hash Collision DoS攻击，基于Java中String的hashcode函数强度很弱，攻击者很容易构造出大量hashcode相同的String对象。然后向服务器提交大量的hashcode相同的字符串参数，那么可以很容易卡死服务器。

# 2. 解决的思路

首先思考下什么样的数据结构可以在插入和查找的时候性能比O(n)好呢？除了哈希外，可能就是平衡树了。Java 8就是在hash冲突的时候，链表的长度达到某个阈值的时候，这个链表就转换成红黑树。而红黑树节点的插入和查找的复杂度为O(logn)，这样就提高了在hash值冲突情况下的HashMap的效率。

![](/img/posts/java8HashMap/java8_hashmap.jpg)

Java8中，当同一个hash值的节点数不小于8时，将不再以单链表的形式存储了，会被调整成一颗红黑树。这就是JDK 7与JDK 8中HashMap实现的最大区别。

# 3. 如何建树

平衡树有一个重要的特性就是左儿子节点的值小于父亲节点，右儿子节点的值大于父亲节点。那么问题来了，我们如何定义要建树节点的大小呢？

有以下2个显而易见的方法可以比较两个Key节点的大小：

1.	Key节点的hashCode：不同hashCode的元素也有可能映射到相同的桶里，我们可以首先根据hashCode来比较两个节点的大小
2.	如果Key节点实现了Comparable接口，可以通过Key的compareTo比较

回想之前讨论的那个Hash Collision DoS攻击，由于Java的String类实现了Comparable接口，因此在Java 8中可以完美解决该攻击，使得服务器不再被卡死。

# 4. 如果Key没有实现Comparable接口呢？

我们自定义的对象很有可能没有实现Comparable接口，在这种情况下树还能平衡么？答案是能平衡。那么怎么保持平衡呢？

通过阅读Java 8的HashMap源码可以发现，在插入的时候如果Key节点的hashCode相同且没有实现Comparable接口，则会调用tieBreakOrder(Object a, Object b)方法来进一步进行比较，比较方法如下：

1.	首先比较两个Key对象的类名
2.	如果相等，再调用System.identityHashCode方法进行比较（System.identityHashCode方法是java根据对象在内存中的地址算出来的一个数值，不同的地址算出来的结果是不一样的）

由以上两个步骤保证了任意两个Key对象都能最终比较出“大小”，保证了树的平衡。

# 5. get方法的问题

get方法查找树的时候，大致过程如下：

1.	要查找的Key和当前节点相等（equals），则返回当前节点的Value
2.	要查找的Key小于当前结点，则访问左子树
3.	要查找的Key大于当前节点，则访问右子树
4.	迭代访问直到当前节点为空，则返回null

那么如何比较两节点之间的大小呢？是否能按找建树的那种比大小的方法来比么？乍一看好像可以，但其实是有问题的：

*	相等的两个节点（equals）地址不一定相同（通过两次new出来的）
*	相等的两个节点（equals）类名不一定相同

举个反例，某个Key没有实现Comparable接口，put了1000个hashCode相同的数据进去，然后又new了一个新的Key（和put进去的某个Key相等），并且通过get方法查询，查出来结果是什么？

通过之前的分析，可得到：

1.	Key节点的hashCode全部相等
2.	Key节点没有实现Comparable接口
3.	Key节点的类名全部相等
4.	剩下的只能比较System.identityHashCode了，结果由于是新new出来的Key，和put进去的Key都不同，所以返回null，不符合预期！

结果真的是这样么？我写了个代码测试了下，结果返回的不是null，是正确的结果，那么上述的分析什么地方的逻辑不太对呢？

通过看Java 8源码得知，get方法访问红黑树时节点之间比较大小如下：

1.	首先比较Key节点的hashCode值
2.	如果hashCode值相等，然后如果实现了Comparable接口，则通过compareTo方法比较
3.	如果compareTo方法比较结果相等，那么遍历整棵子树（源码逻辑时先遍历右子树，后遍历左子树，直到找到相等的节点）

讲到此，应该可以明白get方法符合预期了吧！但是，但是，如果hashCode冲突，并且没有实现Comparable接口，**最坏情况相当于还是遍历了整棵树，也就是说时间复杂度在最差情况下会退化到O(n)**。

# 6. 建议实现Comparable接口

由上述分析不难得出结论：实现一个对象，如果需要把它作为HashMap的Key，则除了实现equals方法和hashCode方法外，**还建议实现Comparable接口**，否则JDK 8对HashMap所做的优化在这个地方起不到最佳效果（只能根据hashCode来建树，如果hashCode相同那就悲剧了）。

最后，源码在put的时候为啥要实现tieBreakOrder这个逻辑呢？我也不知道。