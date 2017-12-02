---
layout: post
title: Java8函数式设计思想示例——拼接用户名
comments: true
author: "Bao Jie"
date: 2017-12-02 00:00:00
header-img: 
tags:
    - Java8
    - 函数式
---

# 1. 场景描述

有这么一种常用的场景：比如从数据库中查询出订单信息，信息中只包含创建订单的用户ID，没有用户名，但前端界面需要展示出用户名，那么就要去用户服务那边根据用户ID拼接用户名，一般代码如下：

	public void fillUserName(List<Order> orders) {
 
		List<Long> userIds = orders.stream()
				.map(Order::getUserId)
				.collect(Collectors.toList());
	 
		List<User> users = userService.findByIds(userIds);
		Map<Long, String> userNameMap = users.stream()
				.collect(Collectors.toMap(User::getId, User::getName));
	 
		for (Order order : orders) {
			String userName = userNameMap.getOrDefault(order.getId(), null);
			order.setUserName(userName);
		}
	}
	
然后接着，又有了一个新模块，叫合同，然后也需要拼接用户名，代码如下：

	public void fillUserName(List<Contract> contracts) {
 
		List<Long> userIds = contracts.stream()
				.map(Contracts::getUserId)
				.collect(Collectors.toList());
	 
		List<User> users = userService.findByIds(userIds);
		Map<Long, String> userNameMap = users.stream()
				.collect(Collectors.toMap(User::getId, User::getName));
	 
		for (Contracts contract : contracts) {
			String userName = userNameMap.getOrDefault(contract.getId(), null);
			contract.setUserName(userName);
		}
	}
	
其实上述两段代码大部分都是相似的，如果之后类似的模块越来越多，那么项目中就会到处散落着这样的代码，导致这块代码逻辑的维护成本非常高（一处改，处处改）。

# 2. 抽象过程

首先看一下fillUserName这个方法实现的主要步骤：

1.	获取用户ID的集合（不同类型的对象获取用户ID的方式不同）
2.	根据用户ID的集合查询对应的用户
3.	将用户集合转成用户ID为Key，用户名为Value的Map
4.	遍历对象，填充用户名（不同类型的对象填充用户名的方式不同）

通过以上步骤可以看出，不同的模块在实现上有以下几个不同点：

1.	处理的对象（一个是订单，一个是合同）
2.	获取用户ID的行为（订单和合同获取用户ID的方式是未知的）
3.	填充用户名的行为（订单和合同填充用户名的方式也是未知的）

因此，我们抽象出来的方法应该包含以上3个参数，其中第一个是对象集合Collection&lt;T&gt; datas，第二个是获取用户ID的函数Function&lt;T, Long&gt; idGetter，第三个是填充用户名的函数BiConsumer&lt;T, String&gt; nameSetter。

# 3. 代码实现

首先在UserService中实现fillUserName抽象方法：

	public <T> void fillUserName(Collection<T> datas, Function<T, Long> idGetter, BiConsumer<T, String> nameSetter) {
 
		List<Long> userIds = datas.stream()
				.map(idGetter)
				.collect(Collectors.toList());
	 
		List<User> users = findByIds(userIds);
		Map<Long, String> userNameMap = users.stream()
				.collect(Collectors.toMap(User::getId, User::getName));
	 
		for (T data : datas) {
			String userName = userNameMap.getOrDefault(idGetter.apply(data), null);
			nameSetter.accept(data, userName);
		}
	}
	
然后Order模块的fillUserName方法实现如下：

	public void fillUserName(List<Order> orders) {
		userService.fillUserName(orders, Order::getUserId, Order::setUserName);
	}
	
Contract模块的fillUserName方法实现如下：

	public void fillUserName(List<Contract> contracts) {
		userService.fillUserName(contracts, Contract::getUserId, Contract::setUserName);
	}
	
现在，如果调用用户服务的逻辑发生变化，只用修改userService中的fillUserName方法就可以了，实现了与各个模块的解耦。

# 4. 总结

总结如下：

*	在出现代码copy现象的时候就应该考虑如何把公共代码抽象出来了
*	抽象过程中，列出实现的主要步骤，甄别出哪些是各个实现都是相同的，不同的地方有哪些？
*	不同的地方区分出是对象还是行为，如果是对象，则通过普通参数传入；如果是行为，则通过函数传入
*	面向对象编程是对对象的抽象，函数式编程则是对行为进行抽象