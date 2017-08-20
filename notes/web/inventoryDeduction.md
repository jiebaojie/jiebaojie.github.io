---
layout: post
notes: true
subtitle: "【技术博客】库存扣多了，到底怎么整"
comments: false
author: "58沈剑"
date: 2017-08-20 00:00:00

---


原文：

[http://zhuanlan.51cto.com/art/201706/542760.htm](http://zhuanlan.51cto.com/art/201706/542760.htm)

[http://zhuanlan.51cto.com/art/201706/542826.htm](http://zhuanlan.51cto.com/art/201706/542826.htm)

*   目录
{:toc }

# 总结

在业务复杂，数据量大，并发量大的情况下，库存扣减容易引发数据的不一致，常见的优化方案有两个：

*	调用“设置库存”接口，能够保证数据的幂等性
*	在实现“设置库存”接口时，需要加上原有库存的比较，才允许设置成功，能解决高并发下库存扣减的一致性问题

# 核心观点

*	用“设置库存”替代“扣减库存”，以保证幂等性
*	使用CAS乐观锁，在“设置库存”时加上原始库存的比对，避免数据不一致

