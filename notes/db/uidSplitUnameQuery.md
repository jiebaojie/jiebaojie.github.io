---
layout: post
notes: true
subtitle: "【技术博客】用uid分库，uname上的查询怎么办？"
comments: false
author: "58沈剑"
date: 2017-12-03 00:00:00

---


原文：[http://zhuanlan.51cto.com/art/201704/537118.htm](http://zhuanlan.51cto.com/art/201704/537118.htm)

*   目录
{:toc }

# 缘起

用户中心是几乎每一个公司必备的基础服务，用户注册、登录、信息查询与修改都离不开用户中心。

当数据量越来越大时，需要多用户中心进行水平切分。最常见的水平切分方式，按照uid取模分库。通过uid取模，将数据分布到多个数据库实例上去，提高服务实例个数，降低单库数据量，以达到扩容的目的。

水平切分之后，uid属性上的查询可以直接路由到库，对于uname上的查询，就不能这么幸运了。

uname上的查询，由于不知道数据落在哪个库上，往往需要遍历所有库【扫全库法】，当分库数量多起来，性能会显著降低。

# 索引表法

思路：uid能直接定位到库，uname不能直接定位到库，如果通过uname能查询到uid，问题解决。解决方案：

*	建立一个索引表记录uname->uid的映射关系
*	用uname来访问时，先通过索引表查询到uid，再定位相应的库
*	索引表属性较少，可以容纳非常多数据，一般不需要分库
*	如果数据量过大，可以通过uname来分库

潜在不足：多一次数据库查询，性能下降一倍

# 缓存映射法

思路：访问索引表性能较低，把映射关系放在缓存里性能更佳

解决方案：

*	uname查询先到cache中查询uid，再根据uid定位数据库
*	假设cache miss，采用扫全库法获取uname对应的uid，放入cache
*	uname到uid的映射关系不会变化，映射关系一旦放入缓存，不会更改，无需淘汰，缓存命中率超高
*	如果数据量过大，可以通过name进行cache水平切分

潜在不足：多一次cache查询

# uname生成uid

思路：不进行远程查询，由uname直接得到uid

解决方案：

*	在用户注册时，设计函数uname生成uid，uid=f(uname)，按uid分库插入数据
*	用uname来访问时，先通过函数计算出uid，即uid=f(uname)再来一遍，由uid路由到对应库

潜在不足：该函数设计需要非常讲究技巧，有uid生成冲突风险

# uname基因融入uid

思路：不能用uname生成uid，可以从uname抽取“基因”，融入uid中

假设分8库，采用uid%8路由，潜台词是，uid的最后3个bit决定这条数据落在哪个库上，这3个bit就是所谓的“基因”。

解决方案：

*	在用户注册时，设计函数uname生成3bit基因，uname_gene=f(uname)
*	同时，生成61bit的全局唯一id，作为用户的标识
*	接着把3bit的uname_gene也作为uid的一部分
*	生成64bit的uid，由id和uname_gene拼装而成，并按照uid分库插入数据
*	用uname来访问时，先通过函数由uname再次复原3bit基因，uname_gene=f(uname)，通过uname_gene%8直接定位到库

# 总结

业务场景：用户中心，数据量大，通过uid分库后，通过uname路由不到库

解决方案：

*	扫全库法：遍历所有库
*	索引表法：数据库中记录uname->uid的映射关系
*	缓存映射法：缓存中记录uname->uid的映射关系
*	uname生成uid
*	uname基因融入uid	