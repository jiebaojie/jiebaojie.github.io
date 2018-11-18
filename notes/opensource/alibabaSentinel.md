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

