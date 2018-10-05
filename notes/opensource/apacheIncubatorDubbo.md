---
layout: post
notes: true
subtitle: "【开源代码】Apache Dubbo"
comments: false
author: "Apache"
date: 2018-10-03 00:00:00

---

GitHub地址：[https://github.com/apache/incubator-dubbo/](https://github.com/apache/incubator-dubbo/)

文档地址：[http://dubbo.apache.org/zh-cn/docs/user/quick-start.html](http://dubbo.apache.org/zh-cn/docs/user/quick-start.html)


*   目录
{:toc }

# 入门

## 背景

随着互联网的发展，网站应用的规模不断扩大，常规的垂直应用架构已无法应对，分布式服务架构以及流动计算架构势在必行，亟需一个治理系统确保架构有条不紊的演进。

![](/img/notes/opensource/apacheIncubatorDubbo/roadmap.jpg)

### 单一应用架构

当网站流量很小时，只需一个应用，将所有功能都部署在一起，以减少部署节点和成本。此时，用于简化增删改查工作量的数据访问框架(ORM)是关键。

### 垂直应用架构

当访问量逐渐增大，单一应用增加机器带来的加速度越来越小，将应用拆成互不相干的几个应用，以提升效率。此时，用于加速前端页面开发的Web框架(MVC)是关键。

### 分布式服务架构

当垂直应用越来越多，应用之间交互不可避免，将核心业务抽取出来，作为独立的服务，逐渐形成稳定的服务中心，使前端应用能更快速的响应多变的市场需求。此时，用于提高业务复用及整合的分布式服务框架(RPC)是关键。

### 流动计算架构

当服务越来越多，容量的评估，小服务资源的浪费等问题逐渐显现，此时需增加一个调度中心基于访问压力实时管理集群容量，提高集群利用率。此时，用于提高机器利用率的资源调度和治理中心(SOA)是关键。

## 需求

Dubbo服务治理

![](/img/notes/opensource/apacheIncubatorDubbo/dubbo_service_governance.jpg)

在大规模服务化之前，应用可能只是通过 RMI 或 Hessian 等工具，简单的暴露和引用远程服务，通过配置服务的URL地址进行调用，通过 F5 等硬件进行负载均衡。

**当服务越来越多时，服务 URL 配置管理变得非常困难，F5 硬件负载均衡器的单点压力也越来越大。** 此时需要一个服务注册中心，动态的注册和发现服务，使服务的位置透明。并通过在消费方获取服务提供方地址列表，实现软负载均衡和 Failover，降低对 F5 硬件负载均衡器的依赖，也能减少部分成本。

**当进一步发展，服务间依赖关系变得错踪复杂，甚至分不清哪个应用要在哪个应用之前启动，架构师都不能完整的描述应用的架构关系。** 这时，需要自动画出应用间的依赖关系图，以帮助架构师理清理关系。

**接着，服务的调用量越来越大，服务的容量问题就暴露出来，这个服务需要多少机器支撑？什么时候该加机器？** 为了解决这些问题，第一步，要将服务现在每天的调用量，响应时间，都统计出来，作为容量规划的参考指标。其次，要可以动态调整权重，在线上，将某台机器的权重一直加大，并在加大的过程中记录响应时间的变化，直到响应时间到达阈值，记录此时的访问量，再以此访问量乘以机器数反推总容量。

以上是 Dubbo 最基本的几个需求。

## 架构

![](/img/notes/opensource/apacheIncubatorDubbo/dubbo_architecture.jpg)

节点角色说明：

| 节点 | 角色说明 |
| ---- | -------- |
| Provider | 暴露服务的服务提供方 |
| Consumer | 调用远程服务的服务消费方 |
| Registry | 服务注册与发现的注册中心 |
| Monitor | 统计服务的调用次数和调用时间的监控中心 |
| Container | 服务运行容器 |

调用关系说明

0.	服务容器负责启动，加载，运行服务提供者。
1.	服务提供者在启动时，向注册中心注册自己提供的服务。
2.	服务消费者在启动时，向注册中心订阅自己所需的服务。
3.	注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。
4.	服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。
5.	服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心。

Dubbo 架构具有以下几个特点，分别是连通性、健壮性、伸缩性、以及向未来架构的升级性。

### 连通性

*	注册中心负责服务地址的注册与查找，相当于目录服务，服务提供者和消费者只在启动时与注册中心交互，注册中心不转发请求，压力较小
*	监控中心负责统计各服务调用次数，调用时间等，统计先在内存汇总后每分钟一次发送到监控中心服务器，并以报表展示
*	服务提供者向注册中心注册其提供的服务，并汇报调用时间到监控中心，此时间不包含网络开销
*	服务消费者向注册中心获取服务提供者地址列表，并根据负载算法直接调用提供者，同时汇报调用时间到监控中心，此时间包含网络开销
*	注册中心，服务提供者，服务消费者三者之间均为长连接，监控中心除外
*	注册中心通过长连接感知服务提供者的存在，服务提供者宕机，注册中心将立即推送事件通知消费者
*	注册中心和监控中心全部宕机，不影响已运行的提供者和消费者，消费者在本地缓存了提供者列表
*	注册中心和监控中心都是可选的，服务消费者可以直连服务提供者

### 健壮性

*	监控中心宕掉不影响使用，只是丢失部分采样数据
*	数据库宕掉后，注册中心仍能通过缓存提供服务列表查询，但不能注册新服务
*	注册中心对等集群，任意一台宕掉后，将自动切换到另一台
*	注册中心全部宕掉后，服务提供者和服务消费者仍能通过本地缓存通讯
*	服务提供者无状态，任意一台宕掉后，不影响使用
*	服务提供者全部宕掉后，服务消费者应用将无法使用，并无限次重连等待服务提供者恢复

### 伸缩性

*	注册中心为对等集群，可动态增加机器部署实例，所有客户端将自动发现新的注册中心
*	服务提供者无状态，可动态增加机器部署实例，注册中心将推送新的服务提供者信息给消费者

### 升级性

当服务集群规模进一步扩大，带动IT治理结构进一步升级，需要实现动态部署，进行流动计算，现有分布式服务架构不会带来阻力。下图是未来可能的一种架构：

![](/img/notes/opensource/apacheIncubatorDubbo/dubbo_architecture_future.jpg)

节点角色说明：

| 节点 | 角色说明 |
| ---- | -------- |
| Deployer | 自动部署服务的本地代理 |
| Repository | 仓库用于存储服务应用发布包 |
| Scheduler | 调度中心基于访问压力自动增减服务提供者 |
| Admin | 统一管理控制台 |
| Registry | 服务注册与发现的注册中心 |
| Monitor | 统计服务的调用次数和调用时间的监控中心 |

## 用法

### 本地服务Spring配置

local.xml:

	<bean id=“xxxService” class=“com.xxx.XxxServiceImpl” />
	<bean id=“xxxAction” class=“com.xxx.XxxAction”>
		<property name=“xxxService” ref=“xxxService” />
	</bean>
	
### 远程服务Spring配置

在本地服务的基础上，只需做简单配置，即可完成远程化：

*	将上面的 local.xml 配置拆分成两份，将服务定义部分放在服务提供方 remote-provider.xml，将服务引用部分放在服务消费方 remote-consumer.xml。
*	并在提供方增加暴露服务配置 <dubbo:service>，在消费方增加引用服务配置 <dubbo:reference>。

remote-provider.xml：

	<!-- 和本地服务一样实现远程服务 -->
	<bean id=“xxxService” class=“com.xxx.XxxServiceImpl” /> 
	<!-- 增加暴露远程服务配置 -->
	<dubbo:service interface=“com.xxx.XxxService” ref=“xxxService” />
	
remote-consumer.xml：

	<!-- 增加引用远程服务配置 -->
	<dubbo:reference id=“xxxService” interface=“com.xxx.XxxService” />
	<!-- 和本地服务一样使用远程服务 -->
	<bean id=“xxxAction” class=“com.xxx.XxxAction”> 
		<property name=“xxxService” ref=“xxxService” />
	</bean>
	
# 快速启动

Dubbo 采用全 Spring 配置方式，透明化接入应用，对应用没有任何 API 侵入，只需用 Spring 加载 Dubbo 的配置即可，Dubbo 基于 Spring 的 Schema 扩展 进行加载。

如果不想使用 Spring 配置，可以通过 API 的方式 进行调用。

## 服务提供者

### 定义服务接口

DemoService.java：

	package com.alibaba.dubbo.demo;

	public interface DemoService {
		String sayHello(String name);
	}
	
### 在服务提供方实现接口

DemoServiceImpl.java：

	package com.alibaba.dubbo.demo.provider;
 
	import com.alibaba.dubbo.demo.DemoService;
	 
	public class DemoServiceImpl implements DemoService {
		public String sayHello(String name) {
			return "Hello " + name;
		}
	}
	
### 用 Spring 配置声明暴露服务

privider.xml：

	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
		xsi:schemaLocation="http://www.springframework.org/schema/beans        http://www.springframework.org/schema/beans/spring-beans-4.3.xsd        http://dubbo.apache.org/schema/dubbo        http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
	 
		<!-- 提供方应用信息，用于计算依赖关系 -->
		<dubbo:application name="hello-world-app"  />
	 
		<!-- 使用multicast广播注册中心暴露服务地址 -->
		<dubbo:registry address="multicast://224.5.6.7:1234" />
	 
		<!-- 用dubbo协议在20880端口暴露服务 -->
		<dubbo:protocol name="dubbo" port="20880" />
	 
		<!-- 声明需要暴露的服务接口 -->
		<dubbo:service interface="com.alibaba.dubbo.demo.DemoService" ref="demoService" />
	 
		<!-- 和本地bean一样实现服务 -->
		<bean id="demoService" class="com.alibaba.dubbo.demo.provider.DemoServiceImpl" />
	</beans>
	
### 加载Spring配置

Provider.java：

	import org.springframework.context.support.ClassPathXmlApplicationContext;
 
	public class Provider {
		public static void main(String[] args) throws Exception {
			ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(new String[] {"http://10.20.160.198/wiki/display/dubbo/provider.xml"});
			context.start();
			System.in.read(); // 按任意键退出
		}
	}
	
## 服务消费者

### 通过Spring配置引用远程服务

consumer.xml：

	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
		xsi:schemaLocation="http://www.springframework.org/schema/beans        http://www.springframework.org/schema/beans/spring-beans-4.3.xsd        http://dubbo.apache.org/schema/dubbo        http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
	 
		<!-- 消费方应用名，用于计算依赖关系，不是匹配条件，不要与提供方一样 -->
		<dubbo:application name="consumer-of-helloworld-app"  />
	 
		<!-- 使用multicast广播注册中心暴露发现服务地址 -->
		<dubbo:registry address="multicast://224.5.6.7:1234" />
	 
		<!-- 生成远程服务代理，可以和本地bean一样使用demoService -->
		<dubbo:reference id="demoService" interface="com.alibaba.dubbo.demo.DemoService" />
	</beans>
	
### 加载Spring配置，并调用远程服务

Consumer.java：

	import org.springframework.context.support.ClassPathXmlApplicationContext;
	import com.alibaba.dubbo.demo.DemoService;
	 
	public class Consumer {
		public static void main(String[] args) throws Exception {
			ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(new String[] {"http://10.20.160.198/wiki/display/dubbo/consumer.xml"});
			context.start();
			DemoService demoService = (DemoService)context.getBean("demoService"); // 获取远程服务代理
			String hello = demoService.sayHello("world"); // 执行远程方法
			System.out.println( hello ); // 显示调用结果
		}
	}
	
注：

1.	该接口需单独打包，在服务提供方和消费方共享
2.	对服务消费方隐藏实现
3.	也可以使用 IoC 注入

# 依赖

## 必须依赖

JDK 1.6+ 

## 缺省依赖

通过 mvn dependency:tree > dep.log 命令分析，Dubbo 缺省依赖以下三方库：

	[INFO] +- com.alibaba:dubbo:jar:2.5.9-SNAPSHOT:compile
	[INFO] |  +- org.springframework:spring-context:jar:4.3.10.RELEASE:compile
	[INFO] |  +- org.javassist:javassist:jar:3.21.0-GA:compile
	[INFO] |  \- org.jboss.netty:netty:jar:3.2.5.Final:compile
	
这里所有依赖都是换照 Dubbo 缺省配置选的，这些缺省值是基于稳定性和性能考虑的。

*	javassist.jar: 如果 &lt;dubbo:provider proxy="jdk" /&gt; 或 &lt;dubbo:consumer proxy="jdk" /&gt;，以及 &lt;dubbo:application compiler="jdk" /&gt;，则不需要。
*	spring-context.jar: 如果用 ServiceConfig 和 ReferenceConfig 的 API 调用，则不需要。
*	netty.jar: 如果 &lt;dubbo:protocol server="mina"/&gt; 或 &lt;dubbo:protocol server="grizzly"/&gt;，则换成 mina.jar 或 grizzly.jar。如果 &lt;protocol name="rmi"/&gt;，则不需要。

## 可选依赖

以下依赖，在主动配置使用相应实现策略时用到，需自行加入依赖。

*	netty-all 4.0.35.Final
*	mina: 1.1.7
*	grizzly: 2.1.4
*	httpclient: 4.5.3
*	hessian_lite: 3.2.1-fixed
*	fastjson: 1.2.31
*	zookeeper: 3.4.9
*	jedis: 2.9.0
*	xmemcached: 1.3.6
*	hessian: 4.0.38
*	jetty: 6.1.26
*	hibernate-validator: 5.4.1.Final
*	zkclient: 0.2
*	curator: 2.12.0
*	cxf: 3.0.14
*	thrift: 0.8.0
*	servlet: 3.0 [5]
*	validation-api: 1.1.0.GA [5:1]
*	jcache: 1.0.0 [5:2]
*	javax.el: 3.0.1-b08 [5:3]
*	kryo: 4.0.1
*	kryo-serializers: 0.42
*	fst: 2.48-jdk-6
*	resteasy: 3.0.19.Final
*	tomcat-embed-core: 8.0.11
*	slf4j: 1.7.25
*	log4j: 1.2.16

注：

1.	理论上 Dubbo 可以只依赖 JDK，不依赖于任何三方库运行，只需配置使用 JDK 相关实现策略
2.	字节码生成
3.	配置解析
4.	网络传输
5.	JEE

# 成熟度

## 功能成熟度

| Feature | Maturity | Strenth | Problem | Advise | User |
| ------- | -------- | ------- | ------- | ------ | ---- |
| 并发控制 | Tested | 并发控制 | | 试用 | |
| 连接控制 | Tested | 连接数控制 | | 试用 | |
| 直连提供者 | Tested | 点对点直连服务提供方，用于测试 | | 测试环境使用 | Alibaba |
| 分组聚合 | Tested | 分组聚合返回值，用于菜单聚合等服务 | 特殊场景使用	| 可用于生产环境 | |
| 参数验证 | Tested | 参数验证，JSR303验证框架集成 | 对性能有影响 | 试用 | LaiWang |
| 结果缓存 | Tested | 结果缓存，用于加速请求 | | 试用 | |
| 泛化引用 | Stable | 泛化调用，无需业务接口类进行远程调用，用于测试平台，开放网关桥接等 | | 可用于生产环境 | Alibaba |
| 泛化实现 | Stable | 泛化实现，无需业务接口类实现任意接口，用于Mock平台 | | 可用于生产环境 | Alibaba |
| 回声测试 | Tested | 回声测试 | | 试用 | |
| 隐式传参 | Stable | 附加参数 | | 可用于生产环境 | |
| 异步调用 | Tested | 不可靠异步调用 | | 试用 | |
| 本地调用 | Tested | 本地调用 | | 试用 | | 
| 参数回调 | Tested | 参数回调 | 特殊场景试用 | 试用 | Registry |
| 事件通知 | Tested | 事件通知，在远程调用执行前后触发 | | 试用 |
| 本地存根 | Stable | 在客户端执行部分逻辑 | | 可用于生产环境 | Alibaba |
| 本地伪装 | Stable | 伪造返回结果，可在失败时执行，或直接执行，用于服务降级 | 需注册中心支持 | 可用于生产环境 | Alibaba |
| 延迟暴露 | Stable | 延迟暴露服务，用于等待应用加载warmup数据，或等待spring加载完成 | | 可用于生产环境 | Alibaba |
| 延迟连接 | Tested | 延迟建立连接，调用时建立 | | 试用 | Registry |
| 粘滞连接 | Tested | 粘滞连接，总是向同一个提供方发起请求，除非此提供方挂掉，再切换到另一台 | | 试用 | Registry |
| 令牌验证 | Tested | 令牌验证，用于服务授权 | 需注册中心支持 | 试用 | |
| 路由规则 | Tested | 动态决定调用关系 | 需注册中心支持 | 试用 | |
| 配置规则 | Tested | 动态下发配置，实现功能的开关 | 需注册中心支持 | 试用 | |
| 访问日志 | Tested | 访问日志，用于记录调用信息 | 本地存储，影响性能，受磁盘大小限制 | 试用 | |
| 分布式事务 | Research | 	JTA/XA三阶段提交事务 | 不稳定 | 不可用 | |

## 策略成熟度

| Feature | Maturity | Strenth | Problem | Advise | User |
| ------- | -------- | ------- | ------- | ------ | ---- |
| Zookeeper注册中心 | Stable | 支持基于网络的集群方式，有广泛周边开源产品，建议使用dubbo-2.3.3以上版本（推荐使用） | 依赖于Zookeeper的稳定性 | 可用于生产环境 | |
| Redis注册中心 | Stable | 支持基于客户端双写的集群方式，性能高 | 要求服务器时间同步，用于检查心跳过期脏数据 | 可用于生产环境 | |
| Multicast注册中心 | Tested | 去中心化，不需要安装注册中心 | 依赖于网络拓扑和路由，跨机房有风险 | 小规模应用或开发测试环境	| |
| Simple注册中心 | Tested | Dogfooding，注册中心本身也是一个标准的RPC服务 | 没有集群支持，可能单点故障 | 试用 | |
| Simple监控中心 | Stable | 支持JFreeChart统计报表 | 没有集群支持，可能单点故障，但故障后不影响RPC运行 | 可用于生产环境 | | 
| Dubbo协议 | Stable | 采用NIO复用单一长连接，并使用线程池并发处理请求，减少握手和加大并发效率，性能较好（推荐使用） | 在大文件传输时，单一连接会成为瓶颈 | 可用于生产环境 | Alibaba |
| Rmi协议 | Stable | 可与原生RMI互操作，基于TCP协议 | 偶尔会连接失败，需重建Stub | 可用于生产环境 | Alibaba |
| Hessian协议 | Stable | 可与原生Hessian互操作，基于HTTP协议 | 需hessian.jar支持，http短连接的开销大 | 可用于生产环境 | |
| Netty Transporter | Stable | JBoss的NIO框架，性能较好（推荐使用） | 一次请求派发两种事件，需屏蔽无用事件 | 可用于生产环境	 | Alibaba |
| Mina Transporter | Stable | 老牌NIO框架，稳定 | 待发送消息队列派发不及时，大压力下，会出现FullGC | 可用于生产环境 | Alibaba |
| Grizzly Transporter | Tested | Sun的NIO框架，应用于GlassFish服务器中 | 线程池不可扩展，Filter不能拦截下一Filter | 试用 | |
| Hessian Serialization | Stable | 性能较好，多语言支持（推荐使用） | Hessian的各版本兼容性不好，可能和应用使用的Hessian冲突，Dubbo内嵌了hessian3.2.1的源码 | 可用于生产环境 | Alibaba |
| Dubbo Serialization | Tested | 通过不传送POJO的类元信息，在大量POJO传输时，性能较好 | 当参数对象增加字段时，需外部文件声明 | 试用 | |
| Json Serialization | Tested | 纯文本，可跨语言解析，缺省采用FastJson解析 | 性能较差 | 试用 | |
| Java Serialization | Stable | Java原生支持 | 性能较差 | 可用于生产环境 | | 
| JavassistProxyFactory | Stable | 通过字节码生成代替反射，性能比较好（推荐使用） | 依赖于javassist.jar包，占用JVM的Perm内存，Perm可能要设大一些：java -XX:PermSize=128m | 可用于生产环境 | Alibaba |
| JdkProxyFactory | Stable | JDK原生支持 | 性能较差 | 可用于生产环境 | |
| Failover Cluster | Stable | 失败自动切换，当出现失败，重试其它服务器，通常用于读操作（推荐使用） | 重试会带来更长延迟 | 可用于生产环境 | Alibaba |
| Failfast Cluster | Stable | 快速失败，只发起一次调用，失败立即报错,通常用于非幂等性的写操作 | 如果有机器正在重启，可能会出现调用失败 | 可用于生产环境 | Alibaba |
| Failsafe Cluster | Stable | 失败安全，出现异常时，直接忽略，通常用于写入审计日志等操作 | 调用信息丢失 | 可用于生产环境 | Monitor |
| Failback Cluster | Tested | 失败自动恢复，后台记录失败请求，定时重发，通常用于消息通知操作 | 不可靠，重启丢失 | 可用于生产环境 | Registry | 
| Forking Cluster | Tested | 并行调用多个服务器，只要一个成功即返回，通常用于实时性要求较高的读操作 | 需要浪费更多服务资源 | 可用于生产环境 | |
| Broadcast Cluster	| Tested | 广播调用所有提供者，逐个调用，任意一台报错则报错，通常用于更新提供方本地状态 | 速度慢，任意一台报错则报错 | 可用于生产环境 | |
| Random LoadBalance | Stable | 随机，按权重设置随机概率（推荐使用） | 在一个截面上碰撞的概率高，重试时，可能出现瞬间压力不均 | 可用于生产环境 | Alibaba |
| RoundRobin LoadBalance | Stable | 轮询，按公约后的权重设置轮询比率 | 存在慢的机器累积请求问题，极端情况可能产生雪崩 | 可用于生产环境 | |
| LeastActive LoadBalance | Stable | 最少活跃调用数，相同活跃数的随机，活跃数指调用前后计数差，使慢的机器收到更少请求 | 不支持权重，在容量规划时，不能通过权重把压力导向一台机器压测容量 | 可用于生产环境 | | 
| ConsistentHash LoadBalance | Stable | 一致性Hash，相同参数的请求总是发到同一提供者，当某一台提供者挂时，原本发往该提供者的请求，基于虚拟节点，平摊到其它提供者，不会引起剧烈变动 | 压力分摊不均 | 可用于生产环境 | |
| 条件路由规则 | Stable | 基于条件表达式的路由规则，功能简单易用 | 有些复杂多分支条件情况，规则很难描述 | 可用于生产环境 | Alibaba |
| 脚本路由规则 | Tested | 基于脚本引擎的路由规则，功能强大 | 没有运行沙箱，脚本能力过于强大，可能成为后门 | 试用 | |
| Spring Container | Stable | 自动加载META-INF/spring目录下的所有Spring配置 | | 可用于生产环境 | Alibaba |
| Jetty Container | Stable | 启动一个内嵌Jetty，用于汇报状态 | 大量访问页面时，会影响服务器的线程和内存 | 可用于生产环境 | Alibaba |
| Log4j Container | Stable | 自动配置log4j的配置，在多进程启动时，自动给日志文件按进程分目录 | 用户不能控制log4j的配置，不灵活 | 可用于生产环境 | Alibaba |