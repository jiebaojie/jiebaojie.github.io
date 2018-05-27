---
layout: post
notes: true
subtitle: "【开源代码】Apache SkyWalking"
comments: false
author: "apache"
date: 2018-05-05 00:00:00

---

GitHub地址：[https://github.com/apache/incubator-skywalking](https://github.com/apache/incubator-skywalking)

中文文档：[https://github.com/apache/incubator-skywalking/blob/master/docs/README_ZH.md](https://github.com/apache/incubator-skywalking/blob/master/docs/README_ZH.md)


*   目录
{:toc }

# 快速入门

## 快速入门——部署步骤

[快速入门——部署步骤](https://github.com/apache/incubator-skywalking/blob/master/docs/cn/Quick-start-CN.md)

### 1. 下载apache-skywalking-apm-incubating-x.y.z.tar.gz 或 apache-skywalking-apm-incubating-x.y.z.zip

### 2. 部署Backend

#### 1). 单机模式

[单机模式](https://github.com/apache/incubator-skywalking/blob/master/docs/cn/Deploy-backend-in-standalone-mode-CN.md)

单机模式默认使用本地H2数据库，不支持集群部署。主要用于：预览、功能测试、演示和低压力系统。

如果使用单机collector用于非演示环境，你可选择使用Elasticsearch作为存储实现。

**在5.0.0-alpha版本中，暂不提供H2实现, 所以在启动之前,必须先部署ElasticSearch**

所需的第三方软件：JDK8+

下载发布版本：前向[发布页面](https://github.com/apache/incubator-skywalking/releases)

Quick-start：Collector单机模拟启动简单，提供和集群模式相同的功能，单机模式下除端口(8080, 10800, 11800, 12800)被占用的情况下，直接启动即可。

部署后台服务：

1.	解压安装包tar -xvf skywalking-collector.tar.gz，windows用户可以选择zip包
2.	运行bin/startup.sh启动。windows用户为.bat文件。

**注意：在5.0.0-alpha 版本中,startup.sh将会启动collector和UI两个进程，UI通过127.0.0.1:10800访问本地collector，无需额外配置。**

使用Elastic Search代替H2存储：在单机模式下，collector也支持其他的存储（当前已支持的ElasticSearch 5.3），取消 application.yml配置文件中的storage 相关配置节的注释，并修改配置,默认的配置是collector 和 Elasticsearch 运行在同一台机器上。

部署Elasticsearch：

*	修改elasticsearch.yml文件
	*	设置 cluster.name: CollectorDBCluster。此名称需要和collector配置文件一致。
	*	设置 node.name: anyname, 可以设置为任意名字，如Elasticsearch为集群模式，则每个节点名称需要不同。
	*	增加如下配置：network.host: 0.0.0.0、thread_pool.bulk.queue_size: 1000
*	启动Elasticsearch

#### 2). 集群模式

[集群模式](https://github.com/apache/incubator-skywalking/blob/master/docs/cn/Deploy-backend-in-cluster-mode-CN.md)

##### 所需的第三方软件

*	被监控程序要求JDK6+
*	SkyWalking collector和WebUI要求JDK8+
*	Elasticsearch 5.x
*	Zookeeper 3.4.10

##### 下载发布版本

*	前向[发布页面](https://github.com/apache/incubator-skywalking/releases)

##### 部署Elasticsearch：

*	修改elasticsearch.yml文件
	*	设置 cluster.name: CollectorDBCluster。此名称需要和collector配置文件一致。
	*	设置 node.name: anyname, 可以设置为任意名字，如Elasticsearch为集群模式，则每个节点名称需要不同。
	*	增加如下配置：network.host: 0.0.0.0、thread_pool.bulk.queue_size: 1000
*	启动Elasticsearch

##### 部署collector

1.	解压安装包tar -xvf skywalking-collector.tar.gz，windows用户可以选择zip包
2.	设置Collector集群模式

集群模式主要依赖Zookeeper的注册和应用发现能力。所以，你只需要调整 config/application.yml中的host和port配置，使用实际IP和端口，代替默认配置。 其次，将storage的注释取消，并修改为Elasticsearch集群的节点地址信息。

config/application.yml

	cluster:
	# 配置zookeeper集群信息
	  zookeeper:
		hostPort: localhost:2181
		sessionTimeout: 100000
	naming:
	# 配置探针使用的host和port
	  jetty:
		host: localhost
		port: 10800
		context_path: /
	remote:
	  gRPC:
		host: localhost
		port: 11800
	agent_gRPC:
	  gRPC:
		host: localhost
		port: 11800
	agent_jetty:
	  jetty:
		host: localhost
		port: 12800
		context_path: /
	analysis_register:
	  default:
	analysis_jvm:
	  default:
	analysis_segment_parser:
	  default:
		buffer_file_path: ../buffer/
		buffer_offset_max_file_size: 10M
		buffer_segment_max_file_size: 500M
	ui:
	  jetty:
		host: localhost
		port: 12800
		context_path: /
	# 配置 Elasticsearch 集群连接信息
	storage:
	  elasticsearch:
		cluster_name: CollectorDBCluster
		cluster_transport_sniffer: true
		cluster_nodes: localhost:9300
		index_shards_number: 2
		index_replicas_number: 0
		ttl: 7
		
3.	运行bin/startup.sh启动。windows用户为.bat文件。

##### 部署UI

1.	解压安装包 tar -xvf skywalking-dist.tar.gz，windows用户可以选择zip包
2.	配置UI集群模式，UI的配置信息保存在 bin/webappService.sh 中 ( windows为bin\webappService.bat)：
	*	server.port：监听端口
	*	collector.ribbon.listOfServers：collector命名服务地址.(与 config/application.yml 中的naming.jetty配置保持相同 )，多个Collector地址以,分割
3.	运行 bin/webappService.sh

### 3. 部署Java Agent

[部署Java Agent](https://github.com/apache/incubator-skywalking/blob/master/docs/cn/Deploy-skywalking-agent-CN.md)

#### 下载skywalking探针发布版本

前往[发布页面](https://github.com/apache/incubator-skywalking/releases)

#### 部署探针

*	拷贝skywalking-agent目录到所需位置，探针包含整个目录，请不要改变目录结构
*	增加JVM启动参数，-javaagent:/path/to/skywalking-agent/skywalking-agent.jar。参数值为skywalking-agent.jar的绝对路径。

新目录结构如下：

	+-- skywalking-agent
		+-- activations
			 apm-toolkit-log4j-1.x-activation.jar
			 apm-toolkit-log4j-2.x-activation.jar
			 apm-toolkit-logback-1.x-activation.jar
			 ...
		+-- config
			 agent.config  
		+-- plugins
			 apm-dubbo-plugin.jar
			 apm-feign-default-http-9.x.jar
			 apm-httpClient-4.x-plugin.jar
			 .....
		skywalking-agent.jar
		
/config/agent.config包含探针所需配置，中文说明如下：

	# 当前的应用编码，最终会显示在webui上。
	# 建议一个应用的多个实例，使用有相同的application_code。请使用英文
	agent.application_code=Your_ApplicationName

	# 每三秒采样的Trace数量
	# 默认为负数，代表在保证不超过内存Buffer区的前提下，采集所有的Trace
	# agent.sample_n_per_3_secs=-1

	# 设置需要忽略的请求地址
	# 默认配置如下
	# agent.ignore_suffix=.jpg,.jpeg,.js,.css,.png,.bmp,.gif,.ico,.mp3,.mp4,.html,.svg

	# 探针调试开关，如果设置为true，探针会将所有操作字节码的类输出到/debugging目录下
	# skywalking团队可能在调试，需要此文件
	# agent.is_open_debugging_class = true

	# 对应Collector的config/application.yml配置文件中 agent_server/jetty/port 配置内容
	# 例如：
	# 单节点配置：SERVERS="127.0.0.1:8080" 
	# 集群配置：SERVERS="10.2.45.126:8080,10.2.45.127:7600" 
	collector.servers=127.0.0.1:10800

	# 日志文件名称前缀
	logging.file_name=skywalking-agent.log

	# 日志文件最大大小
	# 如果超过此大小，则会生成新文件。
	# 默认为300M
	logging.max_file_size=314572800

	# 日志级别，默认为DEBUG。
	logging.level=DEBUG
	
*	启动被监控应用

#### 高级特性

*	插件会被统一放置在plugins目录中，新的插件，也只需要在启动阶段，放在目录中，就自动生效。删除则失效。
*	配置除了通过/config/agent.config文件外，可以通过环境变量和VM参数（-D）来进行设置
	*	参数的key = skywalking. + agent.config文件中的key
	*	优先级：系统环境变量 > VM参数（-D） > /config/agent.config中的配置
*	Log默认使用文件输出，输出到/logs目录中

##### Tomcat配置探针FAQ

Tomcat 7 修改tomcat/bin/catalina.sh，在首行加入如下信息：

	CATALINA_OPTS="$CATALINA_OPTS -javaagent:/path/to/skywalking-agent/skywalking-agent.jar"; export CATALINA_OPTS
	
Tomcat 8 修改tomcat/bin/catalina.sh，在首行加入如下信息：

	set "CATALINA_OPTS=... -javaagent:E:\apache-tomcat-8.5.20\skywalking-agent\skywalking-agent.jar"
	
### 4. 重启并访问系统功能，查看UI即可

## 中间件，框架与类库支持列表

[中间件，框架与类库支持列表](https://github.com/apache/incubator-skywalking/blob/master/docs/Supported-list.md)

*	HTTP Server
	*	Tomcat 7
	*	Tomcat 8
	*	Tomcat 9
	*	Spring Boot Web 4.x
	*	Spring MVC 3.x, 4.x with servlet 3.x
	*	Nutz Web Framework 1.x
	*	Struts2 MVC 2.3.x -> 2.5.x
	*	Resin 3 (Optional)
	*	Resin 4 (Optional)
	*	Jetty Server 9
*	HTTP Client
	*	Feign 9.x
	*	Netflix Spring Cloud Feign 1.1.x, 1.2.x, 1.3.x
	*	Okhttp 3.x
	*	Apache httpcomponent HttpClient 4.2, 4.3
	*	Spring RestTemplete 4.x
	*	Jetty Client 9
	*	Apache httpcomponent AsyncClient 4.x
*	JDBC
	*	Mysql Driver 5.x, 6.x
	*	Oracle Driver (Optional)
	*	H2 Driver 1.3.x -> 1.4.x
	*	Sharding-JDBC 1.5.x
	*	PostgreSQL Driver 8.x, 9.x, 42.x
*	RPC Frameworks
	*	Dubbo 2.5.4 -> 2.6.0
	*	Dubbox 2.8.4
	*	Motan 0.2.x -> 1.1.0
	*	gRPC 1.x
	*	Apache ServiceComb Java Chassis 0.1 -> 0.5,1.0.x
*	MQ
	*	RocketMQ 4.x
	*	Kafka 0.11.0.0 -> 1.0
*	NoSQL
	*	Redis
		*	Jedis 2.x
	*	MongoDB Java Driver 2.13-2.14,3.3+
	*	Memcached Client
		*	Spymemcached 2.x
		*	Xmemcached 2.x
*	Service Discovery
	*	Netflix Eureka
*	Spring Ecosystem
	*	Spring Bean annotations(@Bean, @Service, @Component, @Repository) 3.x and 4.x (Optional)
	*	Spring Core Async SuccessCallback/FailureCallback/ListenableFutureCallback 4.x
*	Hystrix: Latency and Fault Tolerance for Distributed Systems 1.4.20 -> 1.5.12
*	Scheduler
	*	Elastic Job 2.x
*	OpenTracing community supported

Required dependencies for these components must be first manually downloaded before being built, due to license incompatibilities. For this reason these components are not by default included in the SkyWalking releases.

These plugins affect the performance or must be used under some conditions, from experiences. So only released in /optional-plugins, copy to /plugins in order to make them work.

### 如何关闭特定插件

[Disable plugins](https://github.com/apache/incubator-skywalking/blob/master/docs/cn/How-to-disable-plugin-CN.md)

删除plugin目录下的相关jar包：skywalking-agent/plugins/*.jar

	+-- skywalking-agent
		+-- activations
			 apm-toolkit-log4j-1.x-activation.jar
			 apm-toolkit-log4j-2.x-activation.jar
			 apm-toolkit-logback-1.x-activation.jar
			 ...
		+-- config
			 agent.config  
		+-- plugins
			 apm-dubbo-plugin.jar
			 apm-feign-default-http-9.x.jar
			 apm-httpClient-4.x-plugin.jar
			 .....
		skywalking-agent.jar
		
### 可选插件

[可选插件](https://github.com/apache/incubator-skywalking/blob/master/docs/cn/Optional-plugins-CN.md)

可选插件可以由源代码或者agent下的optional-plugins文件夹中提供。

为了使用这些插件，你需要自己编译源代码，或将某些插件复制到/plugins。

#### Spring bean 插件

这个插件允许在Spring上下文中追踪带有@Bean、 @Service、@Component和@Repository注解的bean的所有方法。

为什么这个插件是可选的？ 在Spring上下文中追踪所有方法会创建很多的span，也会消耗更多的CPU，内存和网络。
当然你希望包含尽可能多的span，但请确保你的系统有效负载能够支持这些。

#### Oracle and Resin 插件

由于Oracle和Resin的License，这些插件无法在Apache发行版中提供。

我们应该如何在本地构建这些可选插件？

1.	Resin 3: 下载Resin 3.0.9 并且把jar放在/ci-dependencies/resin-3.0.9.jar。
2.	Resin 4: 下载Resin 4.0.41 并且把jar放在/ci-dependencies/resin-4.0.41.jar。
3.	Oracle: 下载Oracle OJDBC-14 Driver 10.2.0.4.0 并且把jar放在/ci-dependencies/ojdbc14-10.2.0.4.0.jar。

# 高级特性

## 通过系统启动参数进行覆盖配置

[覆盖配置](https://github.com/apache/incubator-skywalking/blob/master/docs/cn/Setting-override-CN.md)

### 版本支持

5.0.0-beta +

探针的覆盖配置从 3.2.5版本就已经支持

### 什么是覆盖配置？

默认情况下, SkyWalking 探针读取agent.config 配置文件, 服务端读取配置文件 application.yml。覆盖配置表示用户可以通过启动参数(-D)来覆盖这些配置文件里面的配置。

### 配置优先级

启动参数配置(-D) > 配置文件

### 覆盖

#### 探针

使用 skywalking. + key 的格式进行配置,覆盖配置文件中的配置。

为什么需要这个前缀? 探针和目标应用共享系统启动参数(环境)的配置，使用这个前缀可以避免变量冲突。

#### Collector

使用配置文件中相同的 key，在启动参数中覆盖collector中的配置。例如:

application.yml的配置：

	agent_gRPC:
	  gRPC:
		host: localhost
		port: 11800
		
在启动脚本中使用如下启动参数配置将端口设置为31200：

	-Dagent_gRPC.gRPC.port=31200
	
## 服务直连(Direct uplink)及禁用名称服务(naming service)

[服务直连(Direct uplink)](https://github.com/apache/incubator-skywalking/blob/master/docs/cn/Direct-uplink-CN.md)

### 版本支持

5.0.0-beta +

### 什么是服务直连(Direct uplink)?

默认情况下, SkyWalking探针使用 名称服务(naming service,即通过名称获取服务地址)的形式获取 collector的地址连接gRPC服务。

**服务直连**意味着子啊名称服务不可用或者低可用的情况下，在探针端直接使用设置的gRPC的地址进行连接。

### 为什么需要这么做？

如果探针通过以下代理上报数据：

1.	私有云(VPCs)
2.	公网(Internet)
3.	不同的子网(subnet)
4.	Ip, Port代理

### 探针配置

1、去掉配置 collector.servers

2、在 agent.config中按照如下配置：

	# Collector agent_gRPC/grpc 地址.
	# 仅仅当不配置的"collector.servers"的时候生效,作为第二种配置地址选择.
	# 如果使用此配置,自动发现服务将无法使用,探针将直接使用此地址进行数据上报.
	# 仅仅当探针端无法连接到`collector`的集群 ip地址时,我们才推荐使用这种配置,比如:
	#   1. 探针和 `collector`在不同的私有云当中.
	#   2. 探针通过外网上报数据到 `collector`.
	# collector.direct_servers=www.skywalking.service.io
	
3、可以只用域名或者IP:PORT形式(逗号分割) 来设置collector.direct_servers.

## 开启TLS

[支持传输层安全TLS(Transport Layer Security)](https://github.com/apache/incubator-skywalking/blob/master/docs/cn/TLS-CN.md)

在通过Internet传输数据时，传输层安全（TLS）是一种非常常见的安全方式 用户可能会在一些场景下遇到这样的情形：

*	被监控(部署探针)的应用中部署在同一个私有云(VPC)区域当中,与此同时，SkyWalking 的服务端部署在另一个私有云(VPC)区域中
*	在这种情况下,就非常有必要做一些传输安全认证

### 配置要求

开启**服务直连**功能，详情参考[文档](https://github.com/apache/incubator-skywalking/blob/master/docs/cn/Direct-uplink-CN.md)

由于通过公网直接上报数据，由于安全问题，名称(naming)服务机制并不适合这种情况。所以我们在HTTP服务的名称服务中不支持TLS。

### 版本支持

5.0.0-beta +

### 认证模式

仅仅支持 **非双向认证**：

*	如果你比较熟悉如何生成 key 文件，可以使用[脚本](https://github.com/apache/incubator-skywalking/blob/master/tools/TLS/tls_key_generate.sh)。
*	在客户端使用 ca.crt文件
*	在服务端使用 server.crt 和 server.pem

### 配置并开启TLS

#### 探针配置

将 ca.crt 放置在探针文件夹的 /ca 文件夹中。需要注意的是，发行的版本中不包含/ca文件夹，需要自行创建。

如果探针检测到文件 /ca/ca.crt，会自动开启 TLS。

#### Collector 配置

agent_gRPC/gRPC 模块支持 TLS。并且现在只有这个模块支持：

*	将application.yml中的 ssl_cert_chain_file 和 ssl_private_key_file 配置打开。
*	ssl_cert_chain_file 配置为 server.crt的绝对路径。
*	ssl_private_key_file 配置为 server.pem的绝对路径。

### 避免端口共享

在大多数情况下，我们建议在agent_gRPC / gRPC和remote / gRPC模块中共享所有gRPC服务的端口。 但是，当你在agent_gRPC / gRPC模块中打开TLS时不要这样做，原因就是无论是否开始TLS,你都无法监听端口。 解决方案, 换一个端口 remote/gRPC/port。

### 其他端口监听如何操作?

请使用其他安全方式确保不能访问 VPC 区域外的其他端口，例如防火墙，代理等。

## 命名空间隔离

[命名空间](https://github.com/apache/incubator-skywalking/blob/master/docs/cn/Namespace-CN.md)

### 版本支持

5.0.0-beta +

### 需求背景

SkyWalking是一个用于从分布式系统收集指标的监控工具。 在实际环境中，一个非常大的分布式系统包括数百个应用程序，数千个应用程序实例。 在这种情况下，更大可能的不止一个组， 甚至还有一家公司正在维护和监控分布式系统。 他们每个人都负责不同的部分，不能共享某些指标。

在这种情况下,命名空间就应运而生了,它用来隔离追踪和监控数据。

### 配置命名空间

#### 在探针配置中配置 agent.namespace

	# The agent namespace
	# agent.namespace=default-namespace
	
默认情况下 agent.namespace 是没有配置的。

默认情况下,SkyWalking 设置的key是 sw3, 更多信息查看文档. 配置好 agent.namespace 之后, key 就被设置为namespace:sw3。

当双方使用不同的名称空间时，跨进程传播链会中断。

#### collector 中设置命名空间

	configuration:
	  default:
		namespace: xxxxx
		
影响：

1.	如果使用 zookeeper开启了集群模式，zookeeper的路径会变为带有命名空间前缀的的路径。
2.	如果使用Elasticsearch 进行存储，所有的type 名称会带有命名空间的前缀。

## 基于Token认证

[基于Token认证](https://github.com/apache/incubator-skywalking/blob/master/docs/cn/Token-auth-CN.md)

### 版本支持

5.0.0-beta +

### 在使用了TLS 认证之后,为何还需要基于 Token 的认证?

TLS 是保证传输层的安全,保证传输的网络是可信的. 基于 token 的认证是为了保证应用的监控数据是 **可信的**。

### Token

在现在的版本中, Token是一个简单的字符串。

#### 设置 Token

在 agent.config 文件中设置 Token：

	# Authentication active is based on backend setting, see application.yml for more details.
	agent.authentication = xxxx
	
在 application.yml 文件中设置 token：

	agent_gRPC:
	  gRPC:
		host: localhost
		port: 11800

		#Set your own token to active auth
		authentication: xxxxxx
		
### 认证失败

collector验证来自探针的每个请求，只有 token 正确，验证才能通过。

如果token不正确，您将在探针端的日志看到如下日志：

	org.apache.skywalking.apm.dependencies.io.grpc.StatusRuntimeException: PERMISSION_DENIED
	
### FAQ

#### 我可以只使用token认证而不用TLS?

不行。从技术层面来说，当然可以。但是token 和 TLS 用于不被信任的网络环境。在这种情况下，TLS显得更加重要，token 认证仅仅在 TLS 认证的之后才能被信任，如果在一个没有 TLS 的网络环节中，token非常容易被拦截和窃取.

#### 现在skywalking是否支持其他的认证机制? 比如 ak/sk?

现在还不支持，但是如果有人愿意提供这些这些新特性，我们表示感谢。

# 孵化特性

## 个性化服务过滤

[个性化服务过滤](https://github.com/apache/incubator-skywalking/blob/master/apm-sniffer/optional-plugins/trace-ignore-plugin/README_CN.md)

提供了一个可选插件 apm-trace-ignore-plugin

### 介绍

*	这个插件的作用是对追踪的个性化服务过滤。
*	你可以设置多个需要忽略的路径, 意味着包含这些路径的追踪信息不会被agent发送到 collector。
*	当前的路径匹配规则是 Ant Path匹配风格 , 例如 /path/*, /path/**, /path/?。
*	将apm-trace-ignore-plugin-x.jar拷贝到agent/plugins后，重启探针即可生效。
*	[Skywalking-使用可选插件 apm-trace-ignore-plugin](https://blog.csdn.net/u013095337/article/details/80452088) 有详细使用介绍

### 如何配置路径

有两种配置方式，可使用任意一种，配置生效的优先级从高到低：

1.	在系统环境变量中配置，你需要在系统变量中添加skywalking.trace.ignore_path, 值是你需要忽略的路径，多个以,号分隔
2.	将/agent/optional-plugins/apm-trace-ignore-plugin/apm-trace-ignore-plugin.config 复制或剪切到 /agent/config/ 目录下，加上配置

trace.ignore_path=/your/path/1/**,/your/path/2/**


# APM相关介绍资料

## OpenTracing中文版

### OpenTracing语义标准

[OpenTracing语义标准](https://opentracing-contrib.github.io/opentracing-specification-zh/specification.html)

#### 综述

这是正式的OpenTracing语义标准。OpenTracing是一个跨编程语言的标准，此文档会避免具有语言特性的概念。比如，我们在文档中使用”interface”，因为所有的语言都包含”interface”这种概念。

##### 版本命名策略

OpenTracing标准使用Major.Minor版本命名策略（即：大版本.小版本），但不包含.Patch版本（即：补丁版本）。如果标准做出不向前兼容的改变，则使用“主版本”号提升。如果是向前兼容的改进，则进行小版本号提升，例如加入新的标准tag, log和SpanContext引用类型。

#### OpenTracing数据模型

OpenTracing中的Trace（调用链）通过归属于此调用链的Span来隐性的定义。 特别说明，一条Trace（调用链）可以被认为是一个由多个Span组成的有向无环图（DAG图）， Span与Span的关系被命名为References。

**译者注: Span，可以被翻译为跨度，可以被理解为一次方法调用, 一个程序块的调用, 或者一次RPC/数据库访问。只要是一个具有完整时间周期的程序访问，都可以被认为是一个span.在此译本中，为了便于理解，Span和其他标准内声明的词汇，全部不做名词翻译。**

每个Span包含以下的状态:（译者注：由于这些状态会反映在OpenTracing API中，所以会保留部分英文说明）

*	An operation name，操作名称
*	A start timestamp，起始时间
*	A finish timestamp，结束时间
*	Span Tag，一组键值对构成的Span标签集合。键值对中，键必须为string，值可以是字符串，布尔，或者数字类型。
*	Span Log，一组span的日志集合。 每次log操作包含一个键值对，以及一个时间戳。 键值对中，键必须为string，值可以是任意类型。 但是需要注意，不是所有的支持OpenTracing的Tracer,都需要支持所有的值类型。
*	SpanContext，Span上下文对象 (下面会详细说明)
*	References(Span间关系)，相关的零个或者多个Span（Span间通过SpanContext建立这种关系）

每一个SpanContext包含以下状态：

*	任何一个OpenTracing的实现，都需要将当前调用链的状态（例如：trace和span的id），依赖一个独特的Span去跨进程边界传输
*	Baggage Items，Trace的随行数据，是一个键值对集合，它存在于trace中，也需要跨进程边界传输

##### Span间关系

一个Span可以与一个或者多个SpanContexts存在因果关系。OpenTracing目前定义了两种关系：ChildOf（父子） 和 FollowsFrom（跟随）。这两种关系明确的给出了两个父子关系的Span的因果模型。 将来，OpenTracing可能提供非因果关系的span间关系。（例如：span被批量处理，span被阻塞在同一个队列中，等等）。

*	ChildOf 引用: 一个span可能是一个父级span的孩子，即”ChildOf”关系。在”ChildOf”引用关系下，父级span某种程度上取决于子span。下面这些情况会构成”ChildOf”关系：
	*	一个RPC调用的服务端的span，和RPC服务客户端的span构成ChildOf关系
	*	一个sql insert操作的span，和ORM的save方法的span构成ChildOf关系
	*	很多span可以并行工作（或者分布式工作）都可能是一个父级的span的子项，他会合并所有子span的执行结果，并在指定期限内返回
*	FollowsFrom 引用: 一些父级节点不以任何方式依赖他们子节点的执行结果，这种情况下，我们说这些子span和父span之间是”FollowsFrom”的因果关系。”FollowsFrom”关系可以被分为很多不同的子类型，未来版本的OpenTracing中将正式的区分这些类型

#### OpenTracing API

OpenTracing标准中有三个重要的相互关联的类型，分别是Tracer, Span 和 SpanContext。下面，我们分别描述每种类型的行为，一般来说，每个行为都会在各语言实现层面上，会演变成一个方法，而实际上由于方法重载，很可能演变成一系列相似的方法。

当我们讨论“可选”参数时，需要强调的是，不同的语言针对可选参数有不同理解，概念和实现方式 。例如，在Go中，我们习惯使用”functional Options”，而在Java中，我们可能使用builder模式。

Tracer：Tracer接口用来创建Span，以及处理如何处理Inject(serialize) 和 Extract (deserialize)，用于跨进程边界传递。它具有如下官方能力

##### 创建一个新Span

必填参数：

*	operation name, 操作名, 一个具有可读性的字符串，代表这个span所做的工作（例如：RPC方法名，方法名，或者一个大型计算中的某个阶段或子任务）。操作名应该是一个抽象、通用，明确、具有统计意义的名称。因此，"get_user" 作为操作名，比 "get_user/314159"更好。

例如，假设一个获取账户信息的span会有如下可能的名称：

| 操作名 | 指导意见 |
| ------ | -------- |
| get    | 太抽象   |
| get_account/792 | 太明确 |
| get_account | 正确的操作名，关于account_id=792的信息应该使用Tag操作 |

可选参数：

*	零个或者多个关联（references）的SpanContext，如果可能，同时快速指定关系类型，ChildOf 还是 FollowsFrom。
*	一个可选的显性传递的开始时间；如果忽略，当前时间被用作开始时间。
*	零个或者多个tag。

返回值：返回一个已经启动Span实例（已启动，但未结束。译者注：英语上started和finished理解容易混淆）

##### 将SpanContext上下文Inject（注入）到carrier

必填参数：

*	SpanContext实例
*	format（格式化）描述，一般会是一个字符串常量，但不做强制要求。通过此描述，通知Tracer实现，如何对SpanContext进行编码放入到carrier中。
*	carrier，根据format确定。Tracer实现根据format声明的格式，将SpanContext序列化到carrier对象中。

##### 将SpanContext上下文从carrier中Extract（提取）

必填参数：

*	format（格式化）描述，一般会是一个字符串常量，但不做强制要求。通过此描述，通知Tracer实现，如何从carrier中解码SpanContext。
*	carrier，根据format确定。Tracer实现根据format声明的格式，从carrier中解码SpanContext。

返回值：返回一个SpanContext实例，可以使用这个SpanContext实例，通过Tracer创建新的Span。

注意：对于Inject（注入）和Extract（提取），format是必须的。

Inject（注入）和Extract（提取）依赖于可扩展的format参数。format参数规定了另一个参数”carrier”的类型，同时约束了”carrier”中SpanContext是如何编码的。所有的Tracer实现，都必须支持下面的format。

*	Text Map: 基于字符串：字符串的map,对于key和value不约束字符集。
*	HTTP Headers: 适合作为HTTP头信息的，基于字符串：字符串的map。（RFC 7230.在工程实践中，如何处理HTTP头具有多样性，强烈建议tracer的使用者谨慎使用HTTP头的键值空间和转义符）
*	Binary: 一个简单的二进制大对象，记录SpanContext的信息。

##### 通过Span获取SpanContext

不需要任何参数。

返回值：Span构建时传入的SpanContext。这个返回值在Span结束后(span.finish())，依然可以使用。

##### 复写操作名（operation name）

必填参数：

*	新的操作名operation name，覆盖构建Span时，传入的操作名。

##### 结束Span

可选参数：

*	一个明确的完成时间;如果省略此参数，使用当前时间作为完成时间。

##### 为Span设置tag

必填参数：

*	tag key，必须是string类型
*	tag value，类型为字符串，布尔或者数字

##### Log结构化数据

必填参数：

*	一个或者多个键值对，其中键必须是字符串类型，值可以是任意类型。某些OpenTracing实现，可能支持更多的log值类型。

可选参数：

*	一个明确的时间戳。如果指定时间戳，那么它必须在span的开始和结束时间之内。

##### 设置一个baggage（随行数据）元素

Baggage元素是一个键值对集合，将这些值设置给给定的Span，Span的SpanContext，以及所有和此Span有直接或者间接关系的本地Span。 也就是说，baggage元素随trace一起保持在带内传递。（译者注：带内传递，在这里指，随应用程序调用过程一起传递）

Baggage元素为OpenTracing的实现全栈集成，提供了强大的功能 （例如：任意的应用程序数据，可以在移动端创建它，显然的，它会一直传递了系统最底层的存储系统。由于它如此强大的功能，他也会产生巨大的开销，请小心使用此特性。

再次强调，请谨慎使用此特性。每一个键值都会被拷贝到每一个本地和远程的下级相关的span中，因此，总体上，他会有明显的网络和CPU开销。

必填参数：

*	baggage key, 字符串类型
*	baggage value, 字符串类型

#####

获取一个baggage元素

必填参数：

*	baggage key, 字符串类型

返回值：相应的baggage value，或者可以标识元素值不存在的返回值（译者注：如Null）。

##### SpanContext

相对于OpenTracing中其他的功能，SpanContext更多的是一个“概念”。也就是说，OpenTracing实现中，需要重点考虑，并提供一套自己的API。 

OpenTracing的使用者仅仅需要，在创建span、向传输协议Inject（注入）和从传输协议中Extract（提取）时，使用SpanContext和references，OpenTracing要求，SpanContext是不可变的，目的是防止由于Span的结束和相互关系，造成的复杂生命周期问题。

##### 遍历所有的baggage元素

遍历模型依赖于语言，实现方式可能不一致。在语义上，要求调用者可以通过给定的SpanContext实例，高效的遍历所有的baggage元素

##### NoopTracer

所有的OpenTracing API实现，必须提供某种方式的NoopTracer实现。NoopTracer可以被用作控制或者测试时，进行无害的inject注入（等等）。例如，在 OpenTracing-Java实现中，NoopTracer在他自己的模块中。

#### 可选API元素

有些语言的OpenTracing实现，为了在串行处理中，传递活跃的Span或SpanContext，提供了一些工具类。例如，opentracing-go中，通过context.Context机制，可以设置和获取活跃的Span。

### 语义惯例

[语义惯例](https://opentracing-contrib.github.io/opentracing-specification-zh/semantic_conventions.html)

OpenTracing标准 描述的语言无关的数据模型，以及OpenTracing API的使用方法。在此数据模型中，包含了两个相关的概念 Span Tag 和 (结构化的) Log Field，尽管在标准中，已经明确了这些操作，但没有定义Span的tag和logging操作时，key的使用规范。

这些语义习惯通过这篇文档进行描述。这篇文档包括两个部分：一. 通过表格罗列出所有的tag和logging操作时，标准的key值。二.描述在特定的典型场景中，如何组合使用这些标准的key值，进行建模。

#### 标准的Span tag 和 log field

##### Span tag清单

Span的tag作用于 整个Span，也就是说，它会覆盖Span的整个事件周期，所以无需指定特别的时间戳。对于由时间点特性的事件，最好使用Span的log操作进行记录。（在本文档的下一章中进行描述）。

| Span tag名称 | 类型   | 描述与实例 |
| ------------ | ----   | ---------- |
| component    | string | 生成此Span所相关的软件包，框架，类库或模块。如 "grpc", "django", "JDBI" |
| db.instance  | string | 数据库实例名称。以Java为例，如果 jdbc.url="jdbc:mysql://127.0.0.1:3306/customers"，实例名为 "customers" |
| db.statement | string | 一个针对给定数据库类型的数据库访问语句。例如， 针对数据库类型 db.type="sql"，语句可能是 "SELECT * FROM wuser_table"; 针对数据库类型为 db.type="redis"，语句可能是 "SET mykey 'WuValue'" |
| db.type      | string | 数据库类型。对于任何支持SQL的数据库，取值为 "sql". 否则，使用小写的数据类型名称，如 "cassandra", "hbase", or "redis" |
| db.user      | string | 访问数据库的用户名。如 "readonly_user" 或 "reporting_user" |
| error        | bool   | 设置为true，说明整个Span失败。译者注：Span内发生异常不等于error=true，这里由被监控的应用系统决定 |
| http.method  | string | Span相关的HTTP请求方法。例如 "GET", "POST" |
| http.status_code | integer | Span相关的HTTP返回码。例如 200, 503, 404 |
| http.url     | string | 被处理的trace片段锁对应的请求URL。 例如 "https://domain.net/path/to?resource=here" |
| message_bus.destination | string | 消息投递或交换的地址。例如，在Kafka中，在生产者或消费者两端，可以使用此tag来存储"topic name" |
| peer.address | string | 远程地址。 适合在网络调用的客户端使用。存储的内容可能是"ip:port"， "hostname"，域名，甚至是一个JDBC的连接串，如 "mysql://prod-db:3306" |
| peer.hostname | string | 远端主机名。例如 "opentracing.io", "internal.dns.name" |
| peer.ipv4    | string | 远端 IPv4 地址，使用 . 分隔。例如 "127.0.0.1" |
| peer.ipv6    | string | 远程 IPv6 地址，使用冒号分隔的元祖，每个元素为4位16进制数。例如 "2001:0db8:85a3:0000:0000:8a2e:0370:7334" |
| peer.port    | integer | 远程端口。如 80 |
| peer.service | string | 远程服务名（针对没有被标准化定义的"service"）。例如 "elasticsearch", "a_custom_microservice", "memcache" |
| sampling.priority | integer | 如果大于0，Tracer实现应该尽可能捕捉这个调用链。如果等于0，则表示不需要捕捉此调用链。如不存在，Tracer使用自己默认的采样机制 |
| span.kind    | string | 基于RPC的调用角色，"client" 或 "server". 基于消息的调用角色，"producer" 或 "consumer" |

##### Log field清单

每个Span的log操作，都具有一个特定的时间戳（这个时间戳必须在Span的开始时间和结束时间之间），并包含一个或多个 field。下面是标准的field。

| Span log field名称 | 类型   | 描述和实例 |
| ------------------ | ------ | ---------- |
| error.kind         | string | 错误类型（仅在event="error"时使用）。如 "Exception", "OSError" |
| error.object       | object | 如果当前语言支持异常对象（如 Java, Python），则为实际的Throwable/Exception/Error对象实例本身。例如 一个 java.lang.UnsupportedOperationException 实例, 一个python的 exceptions.NameError 实例 |
| event              | string | Span生命周期中，特定时刻的标识。例如，一个互斥锁的获取与释放，或 在Performance.timing 规范中描述的，浏览器页面加载过程中的各个事件。 还例如，Zipkin中 "cs", "sr", "ss", 或 "cr". 或者其他更抽象的 "initialized" 或 "timed out"。出现错误时，设置为 "error" |
| message            | string | 简洁的，具有高可读性的一行事件描述。如 "Could not connect to backend", "Cache invalidation succeeded" |
| stack              | string | 针对特定平台的栈信息描述，不强制要求与错误相关。如 "File \"example.py\", line 7, in \<module\>\ncaller()\nFile \"example.py\", line 5, in caller\ncallee()\nFile \"example.py\", line 2, in callee\nraise Exception(\"Yikes\")\n" |

#### 典型场景建模

##### RPCs

使用下面tag为RPC调用建模：

*	span.kind: "client" 或 "server"。在Span开始时，设置此tag是十分重要的，它可能影响内部ID的生成。
*	error: RPC调用是否发生错误
*	peer.address, peer.hostname, peer.ipv4, peer.ipv6, peer.port, peer.service: 可选tag。描述RPC的对端信息。（一般只有在无法获取到这些信息时，才不设置这些值）

##### Message Bus

消息服务是一个异步调用，所以消费端的Span和生产端的Span使用 Follows From 关系。

使用下面tag为消息服务建模：

*	message_bus.destination: 上表已描述
*	span.kind: "producer" 或 "consumer". 建议 在span开始时 设置此tag，它可能影响内部ID的生成。
*	peer.address, peer.hostname, peer.ipv4, peer.ipv6, peer.port, peer.service: 可选tag，描述消息服务中broker的地址。（可能在内部无法获取）

##### Database (client) calls

使用下面tag为数据库客户端调用建模：

*	db.type, db.instance, db.user, 和 db.statement: 上表已描述
*	peer.address, peer.hostname, peer.ipv4, peer.ipv6, peer.port, peer.service: 描述数据库信息的可选tag
*	span.kind: "client"

##### Captured errors，捕获错误

OpenTracing中，根据语言的不同，错误可以通过不同的方式来进行描述，有一些field是专门针对错误输出的，其他则不是（例如：event 或 message）

如果存在错误对象，它其中包含栈信息和错误信息，log时使用如下的field：

*	event="error"
*	error.object=<error object instance>

对于其他语言（译者注：不存在上述的错误对象），或上述操作不可行时：

*	event="error"
*	message="..."
*	stack="..." (可选)
*	error.kind="..." (可选)

通过此方案，Tracer实现可以在需要时，获取所需的错误信息。

# Application Toolkit，应用程序工具包

## 概述

[什么是sky-walking应用程序工具包?](https://github.com/apache/incubator-skywalking/blob/master/docs/cn/Application-toolkit-CN.md)

Sky-walking应用程序工具包是一系列的类库，由skywalking团队提供。通过这些类库，你可以在你的应用程序内，访问sky-walking的一些内部信息。

**最为重要的是**，即使你移除skywalking的探针，或者不激活探针，这些类库也不会对应用程序有任何影响，也不会影响性能。

### 工具包提供以下核心能力

1.	将追踪信息和log组件集成，如log4j, log4j2 和 logback
2.	兼容CNCF OpenTracing标准的手动埋点
3.	使用Skywalking专有的标注和交互性API

**注意**: 所有的应用程序工具包都托管在bitray.com/jcenter. 同时请确保你使用的开发工具包和skywalking的agent探针版本一致。