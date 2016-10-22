---
layout: post
notes: true
subtitle: "【技术博客】使用Java8 Optional的正确姿势"
comments: false
author: "Yanbin"
date: 2016-10-22 12:00:00

---


原文：[http://unmi.cc/proper-ways-of-using-java8-optional/](http://unmi.cc/proper-ways-of-using-java8-optional/)

*   目录
{:toc }

# 概述

所以Optional中我们真正可以来的应该是除了 isPresent() 和 get() 的其他方法。

果断用Optional.of(obj)来构造Optional实例的情况：

1.  当我们非常明确将要传给Optional.of(obj)的obj参数不可能为null时
2.  当想为obj断言不为null时，即我们想在万一obj为null立即报告NullPointerException异常，立即修改，而不是隐藏空指针异常时

# 存在即返回，无则提供默认值

    return user.orElse(null);
    return user.orElse(UNKNOWN_USER);

# 存在即返回，无则由函数来产生

    return user.orElseGet(() -> ...);

# 存在才对它做点什么

    user.ifPresent(System.out::println);

# map函数隆重登场

    return user.map(u -> u.getOrders()).orElse(Collections.emptyList);

# 总结

用了 isPresent() 处理NullPointerException 不叫优雅，有了 orElse, orElseGet 等，特别是 map 方法才叫优雅。

filter() 把不符合条件的值变为 empty()，flatMap() 总是与 map() 方法成对的，orElseThrow() 在有值时直接返回，无值时抛出想要的异常。

一句话小结：使用 Optional 时尽量不直接调用 Optional.get() 方法，Optional.isPresent() 更应该被视为一个私有方法，应依赖于其他像 Optional.orElse(), Optional.orElseGet(), Optional.map() 等这样的方法。



