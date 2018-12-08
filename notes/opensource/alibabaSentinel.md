---
layout: post
notes: true
subtitle: "【开源代码】Sentinel：高可用防护的流量管理框架"
comments: false
author: "Alibaba Open Source"
date: 2018-11-18 00:00:00

---

GitHub地址：[https://github.com/alibaba/Sentinel](https://github.com/alibaba/Sentinel)

中文文档地址：[https://github.com/alibaba/Sentinel/wiki/%E4%BB%8B%E7%BB%8D](https://github.com/alibaba/Sentinel/wiki/%E4%BB%8B%E7%BB%8D)

详细文档：[https://github.com/alibaba/Sentinel/wiki/%E4%B8%BB%E9%A1%B5](https://github.com/alibaba/Sentinel/wiki/%E4%B8%BB%E9%A1%B5)

*   目录
{:toc }

# 介绍

## Sentinel是什么？

随着微服务的流行，服务和服务之间的稳定性变得越来越重要。Sentinel 以流量为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。

Sentinel 具有以下特征：

*	**丰富的应用场景：**Sentinel 承接了阿里巴巴近 10 年的双十一大促流量的核心场景，例如秒杀（即突发流量控制在系统容量可以承受的范围）、消息削峰填谷、实时熔断下游不可用应用等。
*	**完备的实时监控：**Sentinel 同时提供实时的监控功能。您可以在控制台中看到接入应用的单台机器秒级数据，甚至 500 台以下规模的集群的汇总运行情况。
*	**广泛的开源生态：**Sentinel 提供开箱即用的与其它开源框架/库的整合模块，例如与 Spring Cloud、Dubbo、gRPC 的整合。您只需要引入相应的依赖并进行简单的配置即可快速地接入 Sentinel。
*	**完善的 SPI 扩展点：**Sentinel 提供简单易用、完善的 SPI 扩展点。您可以通过实现扩展点，快速的定制逻辑。例如定制规则管理、适配数据源等。

Sentinel的开源生态：

![](/img/notes/opensource/alibabaSentinel/sentinel_open_source.png)

Sentinel 分为两个部分:

*	核心库（Java 客户端）不依赖任何框架/库，能够运行于所有 Java 运行时环境，同时对 Dubbo / Spring Cloud 等框架也有较好的支持。
*	控制台（Dashboard）基于 Spring Boot 开发，打包后可以直接运行，不需要额外的 Tomcat 等应用容器。

## Quick Start

下面的例子将展示应用如何三步接入 Sentinel。同时，Sentinel 也提供一个所见即所得的控制台，可以实时监控资源以及管理规则。

### 1. 在应用中引入Sentinel Jar包

**注意:** Sentinel JAR 包仅支持 Java 6 或者以上版本。

如果应用使用 pom 工程，则在 pom.xml 文件中加入以下代码即可：

	<dependency>
		<groupId>com.alibaba.csp</groupId>
		<artifactId>sentinel-core</artifactId>
		<version>x.y.z</version>
	</dependency>

### 2. 定义资源

接下来，把需要控制流量的代码用 Sentinel API SphU.entry("HelloWorld") 和 entry.exit() 包围起来即可。在下面的例子中，我们将 System.out.println("hello wolrd"); 作为资源，用 API 包围起来。参考代码如下:

	public static void main(String[] args) {
		initFlowRules();
		while (true) {
			Entry entry = null;
			try {
			entry = SphU.entry("HelloWorld");
				System.out.println("hello world");
		} catch (BlockException e1) {
			System.out.println("block!");
		} finally {
		   if (entry != null) {
			   entry.exit();
		   }
		}
		}
	}

### 3. 定义规则

接下来，通过规则来指定允许该资源通过的请求次数，例如下面的代码定义了资源"hello world"每秒最多只能通过 20 个请求。

	private static void initFlowRules(){
		List<FlowRule> rules = new ArrayList<FlowRule>();
		FlowRule rule = new FlowRule();
		rule.setResource("HelloWorld");
		rule.setGrade(RuleConstant.FLOW_GRADE_QPS);
		// Set limit QPS to 20.
		rule.setCount(20);
		rules.add(rule);
		FlowRuleManager.loadRules(rules);
	}
	
完成上面 3 步，Sentinel 就能够正常工作了。

### 4. 检查效果

Demo 运行之后，我们可以在日志 ~/logs/csp/metrics.log.xXXX 里看到下面的输出:

	|--timestamp-|------date time----|--resource-|p |block|s |e|rt
	1529998904000|2018-06-26 15:41:44|hello world|20|0    |20|0|0
	1529998905000|2018-06-26 15:41:45|hello world|20|5579 |20|0|728
	1529998906000|2018-06-26 15:41:46|hello world|20|15698|20|0|0
	1529998907000|2018-06-26 15:41:47|hello world|20|19262|20|0|0
	1529998908000|2018-06-26 15:41:48|hello world|20|19502|20|0|0
	1529998909000|2018-06-26 15:41:49|hello world|20|18386|20|0|0

其中 p 代表通过的请求, block 代表被阻止的请求, s 代表成功通过 Sentinel 的请求个数, e 代表用户自定义的异常, rt 代表平均响应时长。

可以看到，这个程序每秒稳定输出 "hello world" 20 次，和规则中预先设定的阈值是一样的。

### 5. 启动Sentinel控制台

Sentinel 同时提供控制台，可以实时监控各个资源的运行情况，并且可以实时地修改限流规则。

## Sentinel基本概念

### 资源

资源是 Sentinel 的关键概念。它可以是 Java 应用程序中的任何内容，例如，由应用程序提供的服务，或由应用程序调用的其它应用提供的服务，甚至可以是一段代码。在接下来的文档中，我们都会用资源来描述代码块。

只要通过 Sentinel API 定义的代码，就是资源，能够被 Sentinel 保护起来。大部分情况下，可以使用方法签名，URL，甚至服务名称作为资源名来标示资源。

### 规则

围绕资源的实时状态设定的规则，可以包括流量控制规则、熔断降级规则以及系统保护规则。所有规则可以动态实时调整。

## Sentinel 功能和设计理念

### 流量控制

#### 什么是流量控制

流量控制在网络传输中是一个常用的概念，它用于调整网络包的发送数据。然而，从系统稳定性角度考虑，在处理请求的速度上，也有非常多的讲究。任意时间到来的请求往往是随机不可控的，而系统的处理能力是有限的。我们需要根据系统的处理能力对流量进行控制。Sentinel 作为一个调配器，可以根据需要把随机的请求调整成合适的形状，如下图所示：

![](/img/notes/opensource/alibabaSentinel/limitflow.gif)

#### 流量控制设计理念

流量控制有以下几个角度:

*	资源的调用关系，例如资源的调用链路，资源和资源之间的关系；
*	运行指标，例如 QPS、线程池、系统负载等；
*	控制的效果，例如直接限流、冷启动、排队等。

Sentinel 的设计理念是让您自由选择控制的角度，并进行灵活组合，从而达到想要的效果。

### 熔断降级

#### 什么是熔断降级

除了流量控制以外，降低调用链路中的不稳定资源也是 Sentinel 的使命之一。由于调用关系的复杂性，如果调用链路中的某个资源出现了不稳定，最终会导致请求发生堆积。这个问题和 Hystrix 里面描述的问题是一样的。

Sentinel 和 Hystrix 的原则是一致的: 当调用链路中某个资源出现不稳定，例如，表现为 timeout，异常比例升高的时候，则对这个资源的调用进行限制，并让请求快速失败，避免影响到其它的资源，最终产生雪崩的效果。

#### 熔断降级设计理念

在限制的手段上，Sentinel 和 Hystrix 采取了完全不一样的方法。

Hystrix 通过线程池的方式，来对依赖(在我们的概念中对应资源)进行了隔离。这样做的好处是资源和资源之间做到了最彻底的隔离。缺点是除了增加了线程切换的成本，还需要预先给各个资源做线程池大小的分配。

Sentinel 对这个问题采取了两种手段:

*	通过并发线程数进行限制

和资源池隔离的方法不同，Sentinel 通过限制资源并发线程的数量，来减少不稳定资源对其它资源的影响。这样不但没有线程切换的损耗，也不需要您预先分配线程池的大小。当某个资源出现不稳定的情况下，例如响应时间变长，对资源的直接影响就是会造成线程数的逐步堆积。当线程数在特定资源上堆积到一定的数量之后，对该资源的新请求就会被拒绝。堆积的线程完成任务后才开始继续接收请求。

*	通过响应时间对资源进行降级

除了对并发线程数进行控制以外，Sentinel 还可以通过响应时间来快速降级不稳定的资源。当依赖的资源出现响应时间过长后，所有对该资源的访问都会被直接拒绝，直到过了指定的时间窗口之后才重新恢复。


### 系统负载保护

Sentinel 同时对系统的维度提供保护。防止雪崩，是系统防护中重要的一环。当系统负载较高的时候，如果还持续让请求进入，可能会导致系统崩溃，无法响应。在集群环境下，网络负载均衡会把本应这台机器承载的流量转发到其它的机器上去。如果这个时候其它的机器也处在一个边缘状态的时候，这个增加的流量就会导致这台机器也崩溃，最后导致整个集群不可用。

针对这个情况，Sentinel 提供了对应的保护机制，让系统的入口流量和系统的负载达到一个平衡，保证系统在能力范围之内处理最多的请求。

## Sentinel是如何工作的

Sentinel 的主要工作机制如下：

*	对主流框架提供适配或者显示的 API，来定义需要保护的资源，并提供设施对资源进行实时统计和调用链路分析。
*	根据预设的规则，结合对资源的实时统计信息，对流量进行控制。同时，Sentinel 提供开放的接口，方便您定义及改变规则。
*	Sentinel 提供实时的监控系统，方便您快速了解目前系统的状态。