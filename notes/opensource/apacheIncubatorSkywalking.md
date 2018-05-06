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