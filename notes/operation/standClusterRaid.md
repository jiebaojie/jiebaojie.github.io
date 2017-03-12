---
layout: post
notes: true
subtitle: "【技术博客】一张“神图”看懂单机/集群/热备/磁盘阵列（RAID）"
comments: false
author: "架构师之路"
date: 2017-03-12 00:00:00

---


原文：[http://www.wtoutiao.com/p/Z7dhj8.html](http://www.wtoutiao.com/p/Z7dhj8.html)

*   目录
{:toc }

# 单机部署（stand-alone）

只有一个饮水机提供服务，服务只部署一份

# 集群部署（cluster）

有多个饮水机同时提供服务，服务冗余部署，每个冗余的服务都对外提供服务，一个服务挂掉时依然可用

# 热备部署（hot-swap）

只有一个桶提供服务，另一个桶stand-by，在水用完时自动热替换，服务冗余部署，只有一个主服务对外提供服务，影子服务在主服务挂掉时顶上

# 磁盘阵列RAID（Redundant Arrays of independent Disks）

## RAID0

存储性能高的磁盘阵列，又称striping，它的原理是，将连续的数据分散到不同的磁盘上存储，这些不同的磁盘能同时并行存取数据（速度块）

## RAID1

安全性高的磁盘阵列，又称mirror，它的原理是，将数据完全复制到另一个磁盘上，磁盘空间利用率只有50%（冗余，数据安全）

## RAID0+1

RAID0和RAID1的综合方案，这也是国企用的比较多的存储方案（速度快，安全性又高，但是很贵）

## RAID5

RAID0和RAID1的折衷方案，读取速度比较快（不如RAID0，因为多存储了校验位），安全性也很高（可以利用校验位恢复数据），空间利用率也不错（不完全复制，只冗余校验位），这也是互联网公司用的比较多的存储方案