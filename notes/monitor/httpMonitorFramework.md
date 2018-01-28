---
layout: post
notes: true
subtitle: "【技术博客】分级告警策略，人性化系统监控？"
comments: false
author: "架构师之路"
date: 2018-01-28 00:00:00

---


原文：[http://www.sohu.com/a/218956786_178889](http://www.sohu.com/a/218956786_178889)

*   目录
{:toc }

# 一、常见http监控玩法

**提问：有哪些常见http监控需求？**

**回答**：常见的http监控需求有两类：

*	html页面监控
*	返回json数据的http接口

**提问：常见的http监控怎么玩？**

**回答**：一般access日志，通过观测以下两个参数来实施告警：

*	http非200状态码
*	http请求响应时间

# 二、常见的http监控存在什么问题？

**提问：常见的http非200状态码，以及响应时间监控有什么弊端？**

**回答**：每个公司都有自己的404页面，这个页面的http状态码是200，且返回速度极快，根本不能代表html页面的真实运行情况，很难起到真正的监控作用。

画外音：不是说http状态码监控没用，相反，http状态码的监控是很有必要的，http状态码404说明系统一定有问题，但http状态码200不能说明系统没有问题。

**提问：http状态码不能说明问题，那什么才能代表http没有问题呢？**

**回答**：每个http都有自己的业务特性。

*	特性一：需要返回特定的页面内容，如一定要返回一个含“家政”字眼的html页面，才是正确的。
*	特性二：需要返回特定的接口内容，例如，RESTful的获取用户信息接口，假设传入uid=123，会传回：{“RET”:”SUCCESS”, “name”:”shenjian”, “uid”:”123”}，即，http://daojia.com/userinfo/get/?uid=123，一定要返回一个含“shenjian”的字符串，才是正确的。

于是乎，得到了可扩展通用**http监控平台（框架）的思路**：不仅仅要监控http状态码，更重要的是，要监控http返回内容的业务特性。

# 三、可扩展通用http监控平台架构细节

![](/img/notes/monitor/httpMonitorFramework/framework.jpeg)

整个http监控平台的架构如上，分为监控平台层，信息管理层，基础服务层。

## 监控平台层

*	http监控中心：实施监控的主程序
*	http监控配置：可扩展的监控项信息管理

监控项核心信息包含：

*	被监控的html页面/RESTful接口属于哪个集群
*	被监控的URL
*	被监控的URL需要传入的数据，包含GET/POST/COOKIE等数据
*	被监控的http返回的数据中必须包含什么业务特性字符串

以58到家官网html为例，监控项核心信息为：

	[http.monitor.item]
	cluster.name : daojia_main
	url : http://daojia.com/
	result : 家政
	
即，访问http://daojia.com/，返回结果必须包含“家政”。

以获取用户信息RESTful接口为例，监控项核心信息为：

	[http.monitor.item]
	cluster.name : daojia_user
	url : http://daojia.com/userinfo/get/
	get.data : uid=123
	post.data : NULL
	cookie.data : NULL
	result : shenjian
	
即，访问http://daojia.com/userinfo/get/?uid=123，返回结果必须包含“shenjian”。

如果要做成平台，需要有一个**监控项管理后台**，来新增/修改/管理监控项。

监控中心，会遍历所有监控项，并发对各个http监控项实施监控。

## 信息管理层

信息管理层又分为：集群信息管理服务，员工信息管理服务，告警策略管理服务。

集群信息管理服务，主要提供这个接口：

	Info Service::getClusterInfo(String clusterName)
	
即，**通过集群名，获取集群信息**。

集群信息有很多，和监控相关的主要有这么几个信息：

*	集群ip列表，每个web-server都应该被监控到
*	集群负责人，如果监控异常，要将告警发给谁

用户信息管理服务，主要提供这个接口：

	Info Service::getYuanGongInfo(String name)
	
即，**通过员工名，获取员工信息**。

员工信息有很多，和监控相关的主要有这么几个信息：

*	员工手机号，邮箱，微信号，钉钉号等通讯信息
*	如果要实现多级告警策略，还需要获取员工部门及leader的相关信息

告警策略管理服务，主要提供这个接口：

	Bool Service::trySendAlarm(String clusterName, String yuangongName, String ip, String url, …)
	
即，**一旦发现接口有异常，尝试发送告警**。

这个尝试发送告警，并不意味着一定会发送短信或者邮件，因为需要实现一系列人性化的告警策略：

*	集群收敛策略，可以通过clusterName去重
*	接口收敛策略，可以通过url去重
*	定时定频策略，可以通过yuangongName去重
*	白天黑夜策略，可以通过告警发送时间实施
*	...

## 基础服务层

进行完告警策略过滤后，如果真实需要发送告警，调用基础服务层的服务发出。

# 四、可扩展通用http监控框架细节

只要能发短信告警，就能整：

*	http监控项信息：通过配置文件搞
*	集群信息：通过配置文件搞
*	员工信息：通过配置文件搞
*	告警策略信息：不搞告警策略了，异常就发短信

![](/img/notes/monitor/httpMonitorFramework/monitor_service.png)

于是乎，http监控框架变成了这个样子，服务都用配置文件代替了：

http监控项配置，monitor-item.config：

	[http.monitor.item]
	cluster.name : daojia_main
	url : http://daojia.com/
	result : 家政
	
	[http.monitor.item]
	cluster.name : daojia_user
	url : http://daojia.com/userinfo/get/
	get.data : uid=123
	post.data : NULL
	cookie.data : NULL
	result : shenjian
	
集群信息配置，cluster-info.config：

	[daojia_main]
	port : 80
	owner.list: shenjian, zhangsan, lisi

	[daojia_user]
	ip.list : ip11, ip22, ip33
	port : 8080
	owner.list: shenjian

员工信息配置，owner-info.config：

	[shenjian]
	email : XX@XX.com
	phone :15912345678

	[zhangsan]
	email : YY@YY.com
	phone :18611220099
	
# 五、http监控框架伪代码

	// 解析配置文件，取出监控项、集群、员工等信息
	Array[monitor-item] A1=Parse(monitor-item.config);
	Array[cluster-info] A2= Parse(cluster-info.config);
	Array[owner-info] A3=Parse(owner-info.config);

	// 遍历所有监控项
	for(each item in A1) {
	
		// 取出监控项的集群名，URL，http数据，结果等信息
		clusterName= item.clusterName;
		url= item.url;
		getData= item.getData;
		postData= item.postData;
		cookieData= item.cookieData;
		result= item.result

		// 由集群名，获取集群信息
		clusterInfo= A2[clusterName];

		// 由集群信息，获取集群ip列表，集群负责人列表
		List<String>ips = clusterInfo.ip;
		List<String>owners = clusterinfo.owner;

		// 集群内的每一个ip实例web-server，都需要监控
		for(each ip in ips) {

			// 根据ip，url，http数据构造请求
			httpClient client = new httpClient(ip, url, getData, postData, cookieData);

			// 获取http请求执行结果
			httpResponse resp = client.execute();

			// 如果返回为200，并且包含监控项里的业务特性结果
			if(resp.code==200&& resp.contain(result)) {

				//正常，继续监控
				continue;
			}

			// 否则，对所有集群负责人发送告警
			for(each owner in owners) {

				// 取出负责人邮箱和手机号
				email =A3[owner].email;
				phone =A3[owner].phone;

				// 发送邮件与短信告警
				sendEmail(email, ip,url, owner);
				sendSM(phone, ip, url,owner);
			}
		}
	}
	
这个框架的扩展性非常好，能很好的通过配置文件扩展。

monitor-item.config，监控项扩展性

*	新增html页面监控，或者json的RESTful接口监控，只需要在配置中增加一个item
*	配置支持url，get，post，cookie等参数拼装任意http监控请求
*	配置支持不同业务逻辑返回不同的result的业务特性检查

cluster-info.config，集群信息扩展性

*	新增集群，只需在配置中增加一个item
*	集群加了一个实例，只需增加一个ip
*	集群加了一个负责人，只需增加一个owner

owner-info.config，负责人信息扩展性

*	新增负责人，只需要在配置中增加一个item
*	换了手机号/邮箱，只需修改相应配置

**调研**，对于**http监控框架**，你的感受是：

*	ca，有点意思，回去整一个
*	框架化，扩展性挺好，我们公司也实现了
*	平台化，我们公司也实现了
*	弱鸡，我们的http监控比这强多了