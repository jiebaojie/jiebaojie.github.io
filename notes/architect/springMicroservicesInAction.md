---
layout: post
notes: true
subtitle: "Spring Microservices IN ACTION"
comments: false
author: "[美] John Camell （陈文辉 译）"
date: 2018-06-27 00:00:00

---

![](/img/notes/architect/springMicroservicesInAction/spring_microservices_in_action.jpg)

*   目录
{:toc }

# 第1章 欢迎迈入云世界，Spring

本章主要内容：

*	了解微服务以及很多公司使用微服务的原因
*	使用Spring、Spring Boot和Spring Cloud来搭建微服务
*	了解云和微服务为什么与基于微服务的应用程序有关
*	构建微服务涉及的不只是构建微服务代码
*	了解基于云的开发的各个组成部分
*	在微服务开发中使用Spring Boot和Spring Cloud

## 1.1 什么是微服务

微服务是一个小的、松耦合的分布式服务。微服务允许将一个大型的应用分解为具有严格职责定义的便于管理的组件。微服务通过将大型代码分解为小型的精确定义的部分，帮助解决大型代码库中传统的复杂问题。在思考微服务时，一个需要信奉的重要概念就是：分解和分离应用程序的功能，使它们完全彼此独立。

微服务架构具有以下特征：

*	应用程序逻辑分解为具有明确定义了职责范围的细粒度组件，这些组件互相协调提供解决方案
*	每个组件都有一个小的职责领域，并且完全独立部署。微服务应该对业务领域的单个部分负责。此外，一个微服务应该可以跨多个应用程序复用。
*	微服务通信基于一些基本的原则（注意，我说的是原则而不是标准），并采用HTTP和JSON（JavaScript Object Notation）这样的轻量级通信协议，在服务消费者和服务提供者之间进行数据交换。
*	服务的底层采用什么技术实现并没有什么影响，因为应用程序始终使用技术中立的协议（JSON是最常见的）进行通信。这意味着构建在微服务之上的应用程序能够使用多种编程语言和技术进行构建。
*	微服务利用其小、独立和分布式的性质，使组织拥有明确责任领域的小型开发团队。这些团队可能为同一个目标工作，如交付一个应用程序，但是每个团队只负责它们在做的服务。

## 1.2 什么是Spring，为什么它与微服务有关

Spring在应用程序的不同的Java类之间充当一个中间人，管理着它们的依赖关系。Spring本质上就是让用户像玩乐高积木一样将自己的代码组装在一起。

Spring Boot是对Spring框架理念重新思考的结果。虽然Spring Boot包含了Spring的核心特性，但它剥离了Spring中的许多“企业”特性，而提供了一个基于Java的、面向REST的微服务框架。只需一些简单的注解，Java开发者就能够快速构建一个可打包和部署的REST微服务，这个微服务并不需要外部的应用容器。

Spring Cloud框架使实施和部署微服务到私有云或公有云变得更加简单。Spring Cloud在一个公共框架之下封装了多个流行的云管理微服务框架，并且让这些技术的使用和部署像为代码添加注解一样简便。

## 1.3 在本书中读者会学到什么

*	微服务是什么以及构建基于微服务的应用程序的设计考虑因素。
*	什么时候不应该构建基于微服务的应用程序
*	如何使用Spring Boot框架来构建微服务
*	支持微服务应用程序的核心运维模式，特别是基于云的应用程序
*	如何使用Spring Cloud来实现这些运维模式
*	如何利用所学到的知识，构建一个部署管道，将服务部署到内部管理的私有云或公有云厂商所提供的环境中

## 1.4 为什么本书与你有关

## 1.5 使用Spring Boot来构建微服务

## 1.6 为什么要改变构建应用的方式

*	复杂性上升
*	客户期待更快速的交付
*	性能和可伸缩性
*	客户期望他们的应用程序可用

悖论：构建高可伸缩性和高度冗余的应用程序。我们需要将应用程序分解成可以互相独立构建和部署的小型服务。如果将应用程序“分解”为小型服务，并将它们从单体制品中转移出来，那么久可以构建具有下面这些特性的系统。

*	灵活性——可以将解耦的服务进行组合和重新安排，以快速交付新的功能
*	有弹性——解耦的服务意味着应用程序不再是单个“泥浆球”，在这种架构中其中一部分应用程序的降级导致整个应用程序失败。
*	可伸缩性——解耦的服务可以轻松地跨多个服务器进行水平分布，从而可以适当地对功能/服务进行伸缩。

为此，当我们开始讨论微服务时，请记住下面一句话：*小型的、简单的和解耦的服务=可伸缩的、有弹性的和灵活的应用程序。*

## 1.7 云到底是什么

云计算有3种基本模式：

*	基础设施即服务（Infrastructure as a Service, IaaS）
*	平台即服务（Platform as a Service, Paas）
*	软件即服务（Software as a Service, Saas）

每个模型种的关键项都是控制：由谁来负责维护基础设施，以及构建应用程序的技术选择是什么？在Iaas模型中，云供应商提供基础设施，但你需要选择技术并构建最终的解决方案；而在SaaS模型中，你就是供应商所提供的服务的被动消费者，无法对技术进行选择，同时也没有任何责任来维护应用程序的基础设施。

新兴的云平台：

*	函数即服务（Functions as a Service, FaaS）：基于FaaS的应用程序会使用像亚马逊的Lambda技术和Google Cloud函数这样的设施，应用会将代码块以“无服务器（serverless）的形式部署，这些代码会完全在云提供商的平台计算设施上运行。使用FaaS平台，无需管理任何服务器基础设施，只需支付执行函数所需的计算周期。
*	容器即服务（Container as a Service, Caas）：使用容器即服务模型，开发人员将微服务作为便携式容器（如Docker）进行构建并部署到云供应商。CaaS是将服务部署在轻量级的虚拟容器中。云供应商会提供运行容器的虚拟服务器，以及用于构建、部署、监控和伸缩容器的综合工具。

需要重点注意的是，使用云计算的FaaS和CaaS模型，开发人员仍然可以构建基于微服务的架构。微服务概念的重点在于构建有限职责的小型服务，并使用基于HTTP的接口进行通信。新兴的云计算平台（如FaaS和CaaS）是部署微服务的替代基础设施机制。

## 1.8 为什么是云和微服务

基于云的微服务的优势是以弹性的概念为中心。云服务供应商允许开发人员在几分种内快速启动新的虚拟机和容器。如果服务器容量需求下降，开发人员可以关闭虚拟服务器，而不会产生任何额外的费用。使用云供应商部署微服务可以显著地提高应用程序的水平可伸缩性（添加更多的服务器和服务实例）。服务器弹性也意味着应用程序可以更具弹性。如果其中一台微服务遇到问题并且处理能力正在不断地下降，那么启动新的服务实例可以让应用程序保持足够长的存活时间，让开发团队能够从容而优雅地解决问题。

用于微服务的常见部署拓扑结构：

*	简化的基础设施管理——IaaS云计算供应商可以让开发人员最有效地控制他们的服务。开发人员可以通过简单的API调用来启动和停止新服务。
*	大规模的水平可伸缩性——IaaS云服务供应商允许开发人员快速简便地启动服务的一个或多个实例。
*	通过地理分布实现高冗余——Iaas供应商必然拥有多个数据中心。

## 1.9 微服务不只是编写代码

编写健壮的服务需要考虑几个主题：

*	大小适当——适当的大小允许快速更改应用程序，并降低整个应用程序中断的总体风险。
*	位置透明——在微服务应用程序中，多个服务实例可以快速启动和关闭时，如何管理服务调用的物理细节？
*	有弹性——如何通过绕过失败的服务，确保采取”快速失败“的方法来保护微服务消费者和应用程序的整体完整性？
*	可重复——如何确保提供的每个新服务实例与生产环境中的所有其他服务实例具有相同的配置和代码库？
*	可伸缩——如何使用异步处理和事件来最小化服务之间的直接依赖关系，并确保可以优雅地扩展微服务？

本书涵盖以下6类微服务模式：

*	核心微服务开发模式
*	微服务路由模式
*	微服务客户端弹性模式
*	微服务安全模式
*	微服务日志记录和跟踪模式
*	微服务构建和部署模式

### 1.9.1 核心微服务开发模式

基本服务设计的主题：

*	服务粒度——如何将业务域分解为微服务，使每个微服务都具有适当成都的职责？
*	通信协议——开发人员如何与服务进行通信？
*	接口设计——如何设计实际的服务接口，便于开发人员进行服务调用？
*	服务的配置管理——如何管理微服务的配置，以便在不同云环境之间移动时，不必更改核心应用程序代码或配置？
*	服务之间的时间处理——如何使用事件解耦微服务，以便最小化服务之间的硬编码依赖关系，并提高应用程序的弹性？

### 1.9.2 微服务路由模式

*	服务发现——如何使微服务变得可以被发现，以便客户端应用程序在不需要将服务的位置硬编码到应用程序的情况下找到它们？如何确保从可用的服务实例池中删除表现不佳的微服务实例？
*	服务路由——如何为所有服务提供单个入口点，以便将安全策略和路由规则统一应用于微服务应用程序中的多个服务和服务实例？如何确保团队中的每位开发人员不必为他们的服务提供自己的服务路由解决方案？

我们可以实现没有服务路由的服务发现，也可以实现服务路由而无需服务发现（尽管这种实现更加困难）

### 1.9.3 微服务客户端弹性模式

4种客户端弹性模式：

*	客户端负载均衡——如何在服务客户端上缓存服务实例的位置，以便对微服务的多个实例的调用负载均衡到该微服务的所有健康实例？
*	断路器模式——如何阻止客户继续调用出现故障的或遭遇性能问题的服务？
*	后备模式——当服务调用失败时，如何提供”插件“机制，允许服务的客户端尝试通过调用微服务之外的其他方法来执行工作？
*	舱壁模式——微服务应用程序使用多个分布式资源来执行工作。如何区分这些调用，以便表现不佳的服务调用不会对应用程序的其他部分产生负面影响？

### 1.9.4 微服务安全模式

3种基本的安全模式：

*	验证——如何确定调用服务的客户端就是它们声称的那个主体？
*	授权——如何确定调用微服务的客户端是否允许执行它们正在进行的操作？
*	凭据管理和传播——如何避免客户端每次都要提供凭据信息才能访问事务中涉及的服务调用？

### 1.9.5 微服务日志记录和跟踪模式

*	日志关联
*	日志聚合
*	微服务跟踪

### 1.9.6 微服务构建和部署模式

微服务架构的核心原则之一是，微服务的每个实例都应该和其他所有实例相同。

不可变基础设施是成功使用微服务架构的关键因素，因为在生产中必须要保证开发人员为特定微服务启动的每个微服务实例与其他微服务实例相同。

*	构建和部署管道——如何创建一个可重复的构建和部署过程，只需一键即可构建和部署到组织中的任何环境？
*	基础设施即代码——如何将服务的基础设施作为可在源代码管理下执行和管理的代码去对待？
*	不可变服务器——一旦创建了微服务镜像，如何确保它在部署之后永远不会更改？
*	凤凰服务器（Phoenix server）——服务器运行的时间越长，就越容易发生配置漂移。如何确保运行微服务的服务器定期被拆卸，并重新创建一个不可变的镜像？

使用这些模式和主题的目的是，在配置漂移影响到上层环境（如交付准备环境或生产环境）之前，尽可能快地公开并消除配置漂移。

## 1.10 使用Spring Cloud构建微服务

*	开发模式
	*	核心微服务模式Spring Boot
	*	配置管理Spring Cloud Config
	*	异步消息处理Spring Cloud Stream
*	路由模式
	*	服务发现模式Spring Cloud/Netflix Eureka
	*	服务路由模式Spring Cloud/Netflix Zuul
*	客户端弹性模式
	*	客户端负载均衡Spring Cloud/Netflix Ribbon
	*	断路器模式Spring Cloud/Netflix Hystrix
	*	后备模式Spring Cloud/Netflix Hystrix
	*	舱壁模式Spring Cloud/Netflix Hystrix
*	构建部署模式
	*	持续集成Travis CI
	*	基础设施即代码Docker
	*	不可变服务器Docker
	*	凤凰服务器Travis CI/Docker
*	日志记录模式
	*	日志关联Spring Cloud Sleuth
	*	日志聚合Spring Cloud Sleuth(与Papertrail)
	*	微服务跟踪Spring Cloud Sleuth/Zipkin
*	安全模式
	*	授权Spring Cloud Security/OAuth2
	*	验证Spring Cloud Security/OAuth2
	*	凭证管理和传播Spring Cloud Security/OAuth2/JWT
	
### 1.10.1 Spring Boot

Spring Boot是微服务实现中使用的核心技术。Spring Boot通过简化构建基于REST的微服务的核心任务，大大简化了微服务开发。Spring Boot还极大地简化了将HTTP类型的动词（GET、PUT、POST和DELETE）应到到URL、JSON协议序列化与Java对象的相互转化，以及将Java异常映射回标准HTTP错误代码的工作。

### 1.10.2 Spring Cloud Config

Spring Cloud Config通过集中式服务来处理应用程序配置数据的管理，因此应用程序配置数据（特别是环境特定的配置数据）与部署的微服务完全分离。这确保了无论启动多少个微服务实例，这些微服务实例始终具有相同的配置。Spring Cloud Config拥有自己的属性管理存储库，也可以与以下开源项目集成：

*	Git——Spring Cloud Config可以与Git支持的存储库集成，并读出存储库中的应用程序的配置数据。
*	Consul——Consul是一种开源的服务发现工具，允许服务实例向该服务注册自己。服务客户端可以向Consul咨询服务实例的位置。Consul还包括可以被Spring Cloud Config使用的基于键值存储的数据库，能够用来存储应用陈旭的配置数据。
*	Eureka——Eureka是一个开源的Netflix项目，像Consul一样，提供类似的服务发现功能。Eureka同样有一个可以被Spring Cloud Config使用的键值数据库。

### 1.10.3 Spring Cloud服务发现

通过Spring Cloud服务发现，开发人员可以从客户端消费的服务中抽象出部署服务器的物理位置（IP或服务器名称）。服务消费者通过逻辑名称而不是物理位置来调用服务器的业务逻辑。Spring Cloud服务发现也处理服务实例的注册和注销（在服务实例启动和关闭时）。Spring Cloud服务发现可以使用Consul和Eureka作为服务发现引擎。

### 1.10.4 Spring Cloud与Netflix Hystrix和Netflix Ribbon

Spring Cloud与Netflix的开源项目进行了大量整合。对于微服务客户端弹性模式，Spring Cloud封装了Netflix Hystrix库和Netflix Ribbon项目，开发人员可以轻松地在微服务中使用它们。

使用Netflix Hystrix库，开发人员可以快速实现服务客户端弹性模式，如断路器模式和舱壁模式。

虽然Netflix Ribbon项目简化了与诸如Eureka这样的服务发现代理的集成，但它也为服务消费者提供了客户端对服务调用的负载均衡。即使在服务发现代理暂时不可用时，客户端也可以继续进行服务调用。

### 1.10.5 Spring Cloud与Netflix Zuul

Spring Cloud使用Netflix Zuul项目为微服务应用程序提供服务路由功能。Zuul时代理服务请求的服务网关，确保在调用目标服务之前，对微服务的所有调用都经过一个”前门“。通过集中的服务调用，开发人员可以强制执行标准服务策略，如安全授权验证、内容过滤和路由规则。

### 1.10.6 Spring Cloud Stream

Spring Cloud Stream（https://cloud.spirng.io/spring-cloud-stream/）是一种可让开发人员轻松地将轻量级消息处理集成到微服务中的支持技术。借助Spring Cloud Stream，开发人员能够构建智能的服务，它可以使用在应用程序中出现的异步事件。此外，使用Spring Cloud Stream可以快速将微服务与消息代理进行整合，如RabbitMQ和Kafka。

### 1.10.7 Spring Cloud Sleuth

Spring Cloud Sleuth允许将唯一跟踪标识符集成到应用程序所使用的HTTP调用和消息通道（RabbitMQ、Apache Kafka）之中。这些跟踪号码（有时称为关联ID或跟踪ID）能够让开发人员在事务流经应用程序中的不同服务时跟踪事务。有了Spring Cloud Sleuth，这些跟踪ID将自动添加到微服务生成的任何日志记录中。

Spring Cloud Sleuth与日志聚合技术工具（如Papertrail）和跟踪工具（如Zipkin）结合时，能够展现出真正的威力。Papertrail是一个基于云的日志记录平台，用于将日志从不同的微服务实时聚合到一个可查询的数据库中。Zipkin可以获取Spring Cloud Sleuth生成的数据，并允许开发人员可视化单个事务涉及的微服务调用流程。

### 1.10.8 Spring Cloud Security

Spring Cloud Security是一个验证和授权框架，可以控制哪些人可以访问服务，以及他们可以用服务做什么。Spring Cloud Security是基于令牌的，允许服务通过验证服务器发出的令牌彼此进行通信。接收调用的每个服务可以检查HTTP中调用中提供的令牌，以确认用户的身份以及用户对该服务的访问权限。

此外，Spring Cloud Security支持JSON Web Token。JSON Web Token(JWT)框架标准化了创建OAuth2令牌的格式，并为创建的令牌进行数字签名提供了标准。

### 1.10.9 代码供应

要实现代码供应，我们将会转移到其他的技术栈。Spring框架是面向应用程序开发的，它（包括Spring Cloud）没有用于创建”构建和部署“管道的工具。要实现一个”构建和部署“管道，开发人员需要使用Travis CI和Docker这两样工具，前者可以作为构建工具，而后者可以构建包含微服务的服务器镜像。

## 1.11 通过示例来介绍Spring Cloud

@EnableCircuitBreaker和@EnableEurekaClient注解：

*	@EnableCircuitBreaker注解告诉Spring微服务，将要在应用程序使用Netflix Hystrix库。
*	@EnableEurekaClient注解告诉微服务使用Eureka服务发现代理去注册它自己，并且将要在代码中使用服务发现去查询远程REST服务端点。

第一次看到使用Hystrix是在声明hello方法时：

	@HystrixCommand(threadPoolKey = "helloThreadPool")
	public String helloRemoteServiceCall(String firstName, String lastName)
	
@HystrixCommand注解做两件事情：

*	第一件事情：在任何时候调用helloRemoteServiceCall方法，该方法不会被直接调用，这个调用会被委派给由Hystrix管理的线程池。如果调用时间太长（默认为1s），Hystrix将接入并中断调用。这是断路器模式的实现。
*	第二件事情时创建一个由Hystrix管理的名为helloThreadPool的线程池。所有对helloRemoteServiceCall方法的调用只会发生在此线程池中，并且将与正在进行的任何其他远程服务调用隔离。

@EnableEurekaClient的存在告诉Spring Boot，在使用REST服务调用时，使用修改过的RestTemplate类（这不是标准的Spring RestTemplate的工作方式）。这个RestTemplate类允许用户传入自己想要调用的服务的逻辑服务ID：

	ResponseEntity<String> restExchange = restTemplate.exchange(http://logical-service-id/name/{firstName}/{lastName}
	
在幕后，RestTemplate类将与Eureka服务进行通信，并查找一个或多个"name"服务实例的实际位置。作为服务的消费者，开发人员的代码永远不需要知道服务的位置。

另外，RestTemplate类使用Netflix的Ribbon库。Ribbon将会检索与服务有关的所有物理端点的列表。每当客户端调用该服务时，它不必经过集中式负载均衡器就可以对客户端上不同服务实例进行轮询（round-robin）。通过消除集中式负载均衡器并将其移动到客户端，可以消除应用程序基础设施中的其他故障点（故障的负载平衡器）。

## 1.12 确保本书的示例是有意义的

## 1.13 小结

*	微服务是非常小的功能部件，负责一个特定的范围领域。
*	微服务并没有行业标准。与其他早期的Web服务协议不同，微服务采用原则导向的方法，并与REST和JSON的概念相一致。
*	编写微服务很容易，但是完全可以将其用于生产则需要额外的深谋远虑。共有几类微服务开发模式，包括核心开发模式、路由模式、客户端弹性模式、安全模式、日志记录和跟踪模式以及构建和部署模式。
*	虽然微服务与语言无关，但本书引入了两个Spring框架，即Spring Boot和Spring Cloud，它们非常有助于构建微服务。
*	Spring Boot用于简化基于REST的JSON微服务的构建，其目标是让用户只需要少量注解，就能够快速构建微服务。
*	Spring Cloud是Netflix和Hashicorp等公司开源技术的集合，它们已经用Spring注解进行了“包装”，从而显著简化了这些服务的设置和配置。

# 第2章 使用Spring Boot构建微服务

本章主要内容：

*	学习微服务的关键特性
*	了解微服务是如何适应云架构的
*	将业务领域分解成一组微服务
*	使用Spring Boot实现简单的微服务
*	掌握基于微服务架构构建应用程序的视角
*	学习什么时候不应该使用微服务

传统的瀑布方法所面临的挑战在于，许多时候，这些项目交付的软件制品的粒度具有以下特点：

*	紧耦合的
*	由漏洞的
*	单体的

基于微服务的架构具有以下特点：

*	有约束的
*	松耦合的
*	抽象的
*	独立的

基于云的应用程序通常有以下特点：

*	拥有庞大而多样化的用户群
*	极高的运行时间要求
*	不均匀的容量需求

成功的微服务开发的基础是从以下3个关键角色的视角开始的：

*	架构师——架构师的工作是看到大局，了解应用程序如何分解为单个微服务，以及微服务如何交互以交付解决方案。
*	软件开发人员——软件开发人员编写代码并详细了解如何将编程语言和该语言的开发框架用于交付微服务。
*	DevOps工程师——DevOps工程师不仅为生产环境而且为所有非生产环境提供服务部署和管理的智慧。DevOps工程师的口号是：保障每个环境中的**一致性**和**可重复性**。

## 2.1 架构师的故事：设计微服务架构

在构建微服务架构时，项目的架构师主要关注以下3个关键任务：

1.	分解业务问题
2.	建立服务粒度
3.	定义服务接口

### 2.1.1 分解业务问题

可以使用以下指导方针将业务问题识别和分解为备选的微服务：

1.	描述业务问题，并聆听用来描述问题的名词
2.	注意动词。动词突出了动作，通常代表问题域的自然轮廓
3.	寻找数据内聚

### 2.1.2 建立服务粒度

构建微服务架构时，粒度的问题很重要，可以采用以下思想来确定正确的解决方案：

1.	开始的时候可以让微服务涉及的范围更广泛一些，然后将其重构到更小的服务
2.	重点关注服务如何相互交互
3.	随着对问题域的理解不断增长，服务的职责将随着时间的推移而改变

如果微服务过于粗粒度，可能会看到以下现象。

*	服务承担过多的职责
*	该服务正在跨大量表来管理数据
*	测试用例太多

如果微服务过于细粒度呢？

*	问题域的一部分微服务像兔子一样繁殖
*	微服务彼此间严重相互依赖
*	微服务称为简单CRUD服务的集合

### 2.1.3 互相交流：定义服务接口

一般来说，可使用以下指导方针思考服务接口设计。

1.	拥抱REST的理念
2.	使用URI来传达意图
3.	请求和响应使用JSON
4.	使用HTTP状态码来传达结果

## 2.2 何时不应该使用微服务

考量因素：

1.	构建分布式系统的复杂性
2.	虚拟服务器/容器散乱
3.	应用程序的类型
4.	数据事务和一致性

### 2.2.1 构建分布式系统的复杂性

微服务架构需要高度的运维成熟度。除非组织愿意投入高分布式应用程序获得成功所需的自动化和运维工作（监控、伸缩），否则不要考虑使用微服务。

### 2.2.2 服务器散乱

必须对做微服务的灵活性与运行所有这些服务器的成本进行权衡。

### 2.2.3 应用程序的类型

如果正在构建小型的、部门级的应用程序或具有较小用户群的应用程序，那么搭建一个分布式模型（如微服务）的复杂性可能太昂贵了，不值得。

### 2.2.4 数据事务和一致性

如果应用程序需要跨多个数据源进行复杂得数据聚合或转换，那么微服务得分布式性质会让这项工作变得很困难。这样的微服务总是承担太多的职责，也可能变得容易受到性能问题的影响。

## 2.3 开发人员的故事：用Spring Boot和Java构建微服务

### 2.3.1 从骨架项目开始

### 2.3.2 引导Spring Boot应用程序：编写引导类

### 2.3.3 构建微服务的入口：Spring Boot控制器

## 2.4 DevOps工程师的故事：构建运行时的严谨性

对于DevOps工程师来说，微服务的设计关乎在投入生产后如何管理服务。编写代码通常是很简单的，而保持代码运行却是困难的。

微服务开发4条原则：

1.	微服务应该是独立的和可独立部署的，多个服务实例可以使用单个软件制品进行启动和拆卸。
2.	微服务应该是可配置的。
3.	微服务实例需要对客户端是透明的。
4.	微服务应该传达它的健康信息，这是云架构的关键部分。

这4条原则可以映射到以下运维生命周期步骤：

*	服务装配——如何打包和部署服务以保证可重复性和一致性，以便相同的服务代码和运行时被完全相同地部署？
*	服务引导——如何将应用程序和环境特定的配置代码与运行时代码分开，以便可以在任何环境中快速启动和部署微服务实例，而无需对配置微服务进行人为干预？
*	服务注册/发现——部署一个新的微服务实例时，如何让新的服务实例可以被其他应用程序客户端发现
*	服务监控——在微服务环境中，由于高可用性需求，同一服务运行多个实例非常常见。从DevOps的角度来看，需要监控微服务实例，并确保绕过微服务中的任何故障，而且状况不佳的服务实例会被拆卸。

12-Factor：

*	代码库——所有应用程序代码和服务器供应信息都应该处于版本控制中。
*	依赖——通过构建工具，如Maven（Java），明确地声明应用程序使用的依赖项。
*	配置——将应用程序配置（特别是特定于环境的配置）与代码分开存储。
*	后端服务——微服务通常通过网络与数据库或消息系统进行通信。
*	构建、发布和运行——保持部署的应用程序的构建、发布和运行完全分开。
*	进程——微服务应该始终是无状态的。
*	端口绑定——微服务在打包的时候应该是完全独立的，可运行的微服务中要包含一个运行时引擎。
*	并发——需要扩大时，不要依赖单个服务中的线程模型。
*	可任意处置——微服务时可任意处置的，可以根据需要启动和停止。
*	开发环境与生产环境等同——最小化服务运行的所有环境（包括开发人员的台式机）之前存在的差距。
*	日志——日志是一个事件流。当日志被写出时，它们应该可以流式传输到诸如Splunk或Fluentd这样的工具，这些工具将整理日志并将它们写入中央位置。微服务不应该关心这种情况发生的机制，开发人员应该在它们被写出来的时候通过标准输出直观地查看日志。
*	管理进程——开发人员通常不得不针对他们的服务执行管理任务（数据移植或转换）。这些任务不应该是临时指定的，而应该通过源代码存储库管理和维护的脚本来完成。

### 2.4.1 服务装配：打包和部署微服务

*	构建和部署引擎将使用Spring Boot的Maven脚本启动构建
*	当开发人员checks in代码时，构建和部署引擎会构建并打包代码
*	构建的输出是一个可执行JAR，它包含嵌入在其中的应用程序和运行时容器

### 2.4.2 服务引导：管理微服务的配置

*	理想情况下，配置存储应能够对所有配置更改进行版本化，并审计跟踪配置数据的最后更改
*	当微服务启动时，任何特定于环境的信息或应用程序配置信息数据都应该是：
	*	作为环境变量传入启动服务
	*	从集中式配置管理存储库中读取数据
*	如果服务的配置发生变化，运行旧配置的服务应该被拆除，或者通知重新读取配置信息

### 2.4.3 服务注册和发现：客户端如何与微服务通信

*	服务实例启动时，它将用服务发现代理注册自己
*	服务客户端永远不知道服务实例所在的物理位置。相反，它会向服务发现代理询问一个健康服务实例的位置。

### 2.4.4 传达微服务的“健康状况”

*	服务发现代理监视服务实例的健康状况。如果实例发生故障，则健康检查从可用实例池中删除它。
*	大多数服务实例将公开一个由服务发现代理调用的健康姜茶URL。如果该调用返回一个HTTP错误，或者没有及时响应，服务发现代理可以关闭该实例，或者不路由它。

## 2.5 将视角综合起来

1.	架构师——专注于业务问题的自然轮廓。描述业务问题域，并听取别人所讲述的故事，按照这种方式，筛选出目标备选微服务。最好从“粗粒度”的微服务开始，并重构到较小的服务，而不是从一大批小型服务开始。微服务架构像大多数优秀的架构一样，是按需调整的，而不是预先计划好的。
2.	软件工程师——尽管服务很小，但并不意味着就应该把良好的设计原则抛于脑后。专注于构建分层服务，服务中的每一层都有离散的职责。避免在代码中构建框架的诱惑，并尝试使每个微服务完全独立。过早的框架设计和采用框架会在应用程序生命周期的后期产生巨大的维护成本。
3.	DevOps工程师——服务不存在于真空中。尽早建立服务的生命周期。DevOps视角不仅要关注如何自动化服务的构建和部署，还要关注如何监控服务的健康状况，并在出现问题时做出反应。

## 2.6 小结

*	要想通过微服务获得成功，需要综合架构师、软件开发人员和DevOps的视角。
*	微服务是一种强大的架构范型，它有优点和缺点。并非所有应用程序都应该是微服务应用程序。
*	从架构师的角度来看，微服务是小型的、独立的和分布式的。微服务应具有狭窄的边界，并管理一小组数据。
*	从开发人员的角度来看，微服务通常使用REST风格的设计构建，JSON作为服务发送和接收数据的净荷。
*	Spring Boot是构建微服务的理想框架，因为它允许开发人员使用几个简单的注解即可构建基于REST的JSON服务。
*	从DevOps的角度来看，微服务如何打包、部署和监控至关重要。
*	开箱即用。Spring Boot允许用户用单个可执行JAR文件交付服务。JAR文件中的嵌入式Tomcat服务器承载该服务。
*	Spring Boot框架附带的Spring Actuator会公开有关服务运行健康状况的信息以及有关服务运行时的信息。

# 第3章 使用Spring Cloud配置服务器控制配置

本章主要内容：

*	将服务配置与服务代码分开
*	配置Spring Cloud配置服务器
*	集成Spring Boot微服务
*	加密敏感属性

基于云的微服务开发强调以下几点：

1.	应用程序的配置与正在部署的实际代码完全分离
2.	构建服务器、应用程序以及一个不可变的镜像，它们在各环境中进行提升时永远不会发生变化
3.	在服务启动时通过环境变量注入应用程序配置信息，或者在微服务启动时通过集中式存储库读取应用程序配置信息

## 3.1 管理配置（和复杂性）

4条原则：

1.	分离——我们希望将服务配置信息与服务的实际物理部署完全分开。
2.	抽象——将访问配置数据的功能抽象到一个服务接口中
3.	集中——将应用程序配置集中在尽可能少的存储库中
4.	稳定——保证其高可用户冗余

### 3.1.1 配置管理架构

1.	微服务实例启动并获取配置信息
2.	实际配置驻留在存储库中
3.	开发人员的更改将通过构建和部署管理推送到配置存储库
4.	通知配置更改的应用程序自行刷新

### 3.1.2 实施选择

用于实施配置管理系统的开源项目：

*	Etcd：使用Go开发的开源项目，用于服务发现和键值管理，使用raft协议作为它的分布式计算模型，特点：
	*	非常快和可伸缩
	*	可分布式
	*	命令行驱动
	*	易于搭建和使用
*	Eureka：由Netflix开发，久经测试，用于服务发现和键值管理，特点：
	*	分布式键值存储
	*	灵活，需要费些功夫去设置
	*	提供开箱即用的动态客户端刷新
*	Consul：由Hashicorp开发，特性上类似于Etcd和Eureka，它的分布式计算模型使用了不同的算法（SWIM协议），特点：
	*	快速
	*	提供本地服务发现功能，可直接与DNS集成
	*	没有提供开箱即用的动态客户端刷新
*	ZooKeeper：一个提供分布式锁定功能的Apache项目，经常用作访问键值数据的配置管理解决方案，特点：
	*	最古老、最久经测试的解决方案
	*	使用最为复杂
	*	可用作配置管理，但只有在其他架构中
	*	已经使用了ZooKeeper的时候才考虑使用它
*	Spring Cloud Config：一个开源项目，提供不同后端支持的通用配置管理解决方案。它可以将Git、Eureka和Consul作为后端进行整合，特点：
	*	非分布式键值存储
	*	提供了对Spring和非Spring服务的紧密集成
	*	可以使用多个后端来存储配置数据，包括共享文件系统，Eureka、Consul和Git
	
使用Spring Cloud配置服务器的原因：

1.	Spring Cloud配置服务器易于搭建和使用
2.	Spring Cloud配置与Spring Boot紧密集成
3.	Spring Cloud配置服务器提供多个后端用于存储配置数据
4.	Spring Cloud配置服务器可以直接与Git源控制平台集成

## 3.2 构建Spring Cloud配置服务器

建议不要在中大型云应用中使用基于文件系统的解决方案。使用文件系统方法，意味着要为想要访问应用程序配置数据的所有云配置服务器实现共享文件挂载点。在云中创建共享文件系统服务器是可行的，但它将维护此环境的责任放在开发人员身上。

### 3.2.1 创建Spring Cloud Config引导类

@EnableConfigServer使服务成为Spring Cloud Config服务

### 3.2.2 使用带有文件系统的Spring Cloud配置服务器

如果使用Spring Cloud Config的本地文件系统版本，那么在本地运行代码时，需要修改spring.cloud.config.server.native.searchLocations属性以反映本地文件路径。

## 3.3 将Spring Cloud Config与Spring Boot客户端集成

将Spring Profile和端点信息传递给许可证服务：

	Spring profile = dev
	Spring cloud config endpoint = http://loocalhost:8888
	
许可证服务联系Spring Cloud配置服务

从存储库检索特定profile的配置信息

### 3.3.1 建立许可证服务对Spring Cloud Config服务器的依赖

	<dependency>
		<groupId>org.springframework.client.cloud</groupId>
		<artifactId>spring-cloud-config-client</artifactId>
	</dependency>
	
### 3.3.2 配置许可证服务以使用Spring Cloud Config

	spring.cloud.config.uri = http://localhost:8888
	
是服务查找Spring Cloud配置服务器端点的位置。

通过Spring Boot Actuator来增强服务的自我检查能力，所以可以通过访问http://localhost:8080/env来确认正在运行的环境。/env端点将提供有关服务的配置信息的完整列表，包括服务启动的属性和端点。

### 3.3.3 使用Spring Cloud配置服务器连接数据源

### 3.3.4 使用@Value注解直接读取属性

虽然可以将配置的值直接诸如各个类的属性中，但将所有配置信息集中到一个配置类，然后将配置类注入需要它的地方是很有用的。

### 3.3.5 使用Spring Cloud配置服务器和Git

*	spring.cloud.config.server.git.uri：属性提供要连接的存储库URL。
*	spring.cloud.config.server.git.searchPaths：属性告诉Spring Cloud Config服务器在云配置服务器启动时应该在Git存储库中搜索的相对路径。

### 3.3.6 使用Spring Cloud配置服务器刷新属性

Spring Boot Actuator提供了一个@RefreshScope注解，允许开发团队访问/refresh端点，这会强制Spring Boot应用程序重新读取应用程序配置。

## 3.4 保护敏感的配置信息

Spring Cloud Config支持使用对称加密（共享密钥）和非对称加密（公钥/私钥）。

搭建Spring Cloud配置服务器以使用对称密钥的加密：

1.	下载并安装加密所需要的Oracle JCE jar
2.	创建加密密钥
3.	加密和解密属性
4.	配置微服务以在客户端使用加密

### 3.4.1 下载并安装加密所需的Oracle JCE jar

首先，需要下载Oracle的不限长度的Java加密扩展（Unlimited Strength Java Cryptography Extension, JCE）。它无法通过Maven下载，必须从Oracle公司下载。下载包含JCE jar的zip文件后，必须执行以下操作：

1.	切换到$JAVA_HOME/jre/lib/security文件夹
2.	将$JAVA_HOME/jre/lib/security目录中的local_policy.jar和US_export_policy.jar文件备份到其他位置。
3.	解压从Oracle下载的JCE zip文件
4.	将local_policy.jar和US_export_policy.jar复制到$JAVA_HOME/jre/lib/security目录中。
4.	配置Spring Cloud Config以使用加密。

### 3.4.2 创建加密密钥

使用Spring Cloud配置服务器，对称加密密钥是通过操作系统环境变量ENCRYPT_KEY传递给服务的字符串。

### 3.4.3 加密和解密属性

在启动Spring Cloud Config实例时，Spring Cloud Config将检测到环境变量ENCRYPT_KEY已设置，并自动将两个新端点（/encrypt和/decrypt）添加到Spring Cloud Config服务。

Spring Cloud配置服务器要求所有已加密的属性前面加上{cipher}。{cipher}告诉Spring Cloud配置服务器它正在处理已加密的值。并使用/default端点时，数据以纯文本形式公开了。

	spring.datasource.password: "{cipher}858201e10fe......."
	
使用/default端点时，数据以纯文本形式公开了。

在默认情况下，Spring Cloud Config将在服务器上解密所有属性，并将未加密的纯文本作为结果传回请求属性的应用程序。但是，开发人员可以告诉Spring Cloud Config不要在服务器上进行解密，并让应用程序负责检索配置数据以解密已加密的属性。

### 3.4.4 配置微服务以在客户端使用加密

要让客户端对属性进行解密，需要做以下3件事情：

1.	配置Spring Cloud Config不要在服务器端解密属性：spring.cloud.config.server.encrypt.enabled=false
2.	在许可证服务器上设置对称密钥。确保ENCRYPT_KEY环境变量与Spring Cloud Config服务器使用的对称密钥相同。
3.	将spring-security-rsa JAR添加到许可证服务的pom.xml文件中。

依赖项：

	<dependency>
		<groupId>org.springframework.security</groupId>
		<artifactId>spring-security-rsa</artifactId>
	</dependency>
	
现在访问/default端点，就会发现数据以加密形式返回。在从Spring Cloud Config加载属性时，该属性将由调用服务解密。

## 3.5 最后的想法

使用基于云的模型，应用程序配置数据应该与程序完全分离，并在运行时注入相应的配置数据，以便在所有环境中一致地提升相同的服务器和应用程序制品。

## 3.6 小结

*	Spring Cloud配置服务器允许使用环境特定值创建应用程序属性。
*	Spring使用Spring profile来启动服务，以确定要从Spring Cloud Config服务检索哪些环境属性。
*	Spring Cloud配置服务可以使用基于文件或基于Git的应用程序配置存储库来存储应用程序属性。
*	Spring Cloud配置服务允许使用对称加密和非对称加密对敏感属性文件进行加密。

# 第4章 服务发现

本章主要内容：

*	为什么服务发现对基于云的应用程序环境很重要
*	与传统的负载均衡方法作对比，了解服务发现的优缺点
*	建立一个Spring Netflix Eureka服务器
*	通过Eureka注册一个基于Spring Boot的微服务
*	使用Spring Cloud和Netflix的Ribbon库来完成客户端负载均衡

服务发现对于微服务和基于云的应用程序至关重要：

*	首先，它为应用团队提供了一种能力，可以快速地对在环境中运行的服务实例数量进行水平伸缩
*	其次，它有助于提高应用程序的弹性。当微服务实例变得不健康或不可用时，大多数服务发现引擎将从内部可用服务列表中移除该实例。

## 4.1 我的服务在哪里

在非云的世界中，这种服务位置解析通常由DNS和网络负载均衡器的组合来解决：

1.	应用程序使用通用DNS和特定于服务的路径来调用服务
2.	负载均衡器查找托管服务的服务器的物理地址
3.	服务部署到应用程序容器中，应用程序容器运行在持久服务器上
4.	辅助负载均衡器检查主负载均衡器，并在必要时进行接管

但对基于云的微服务应用程序来说，这种模型并不适用。原因有以下几个：

*	单点故障
*	有限的水平可伸缩性
*	静态管理
*	复杂

## 4.2 云中的服务发现

基于云的微服务环境的解决方案是使用服务发现机制，这一机制具有以下特点：

*	高可用
*	点对点
*	负载均衡
*	有弹性——本地缓存允许服务发现功能逐步降级
*	容错——服务发现需要检测出服务实例什么时候是不健康的，并从可以接收客户端请求的可用服务列表中移除该实例。

### 4.2.1 服务发现架构

4个概念：

*	服务注册——服务如何使用服务发现代理进行注册？
*	服务地址的客户端查找——服务客户端查找服务信息的方法是什么？
*	信息共享——如何跨节点共享服务信息？
*	健康监测——服务如何将它的健康信息传回给服务发现代理？

流程：

1.	可以通过逻辑名称从服务发现代理查找服务的位置
2.	一个服务上线时，这个服务会向服务发现代理注册它的IP地址
3.	服务发现节点共享服务实例的健康信息
4.	服务向服务发现代理发送心跳包。如果服务死亡，服务发现层将移除“死亡的”实例的IP

一种更健壮的方法是使用所谓的客户端负载均衡。在这个模型中，当服务消费者需要调用一个服务时：

1.	它将联系服务发现服务，获取它请求的所有服务实例，然后在服务消费者的机器上本地缓存数据。
2.	每当客户端需要调用该服务时，服务消费者将从缓存中查找该服务的位置信息。通常，客户端缓存将使用简单的负载均衡算法。
3.	然后，客户端将定期与服务发现服务进行联系，并刷新服务实例的缓存。

如果在调用服务的过程中，服务调用失败，那么本地的服务发现缓存失效，服务发现客户端将尝试从服务发现代理刷新数据。

### 4.2.2 使用Spring和Netflix Eureka进行服务发现实战

通过许可证服务和组织服务实现客户端缓存和Eureka，可以减轻Eureka服务器上的负载，并提高Eureka不可用时的客户端稳定性：

1.	当服务实例启动时，它门将使用Eureka注册它们的IP。
2.	当许可证服务调用组织服务时，它将使用Ribbon来查看组织服务
3.	Ribbon将定期刷新它的IP地址缓存

## 4.3 构建Spring Eureka服务

spirng-cloud-starter-eureka-server：告诉Maven构建包含Eureka库（其中包括Ribbon）

配置：

*	server.port：用于设置Eureka服务的默认端口
*	eureka.client.registerWithEureka=false：告知服务，在Spring Boot Eureka应用程序启动时不要通过Eureka服务注册，因为它本身就是Eureka服务。
*	eureka.client.fetchRegistry=false：Eureka服务启动时，它不会尝试本地缓存注册表信息。
*	eureka.server.waitTimeInMsWhenSyncEmpty：在服务器接收请求之前等待的初始时间

每次服务注册需要30s的时间才能显示在Eureka服务中，因为Eureka需要从服务接收3次连续心跳包ping，每次心跳包ping间隔10s，然后才能使用这个服务。

@EnableEurekaServer：在Spring服务中启用Eureka服务器

## 4.4 通过Spring Eureka注册服务

spring-cloud-starter-eureka：拥有Spring Cloud用于与Eureka服务进行交互的jar文件，以便可以使用Eureka注册服务

配置：

*	spirng.application.name：将使用Eureka注册的服务的逻辑名称（建议放到bootstrp.yml文件中）
*	eureka.instance.preferIpAddress=true：注册服务的IP，而不是服务器名称
*	eureka.client.registerWithEureka=true：向Eureka注册服务
*	eureka.client.fetchRegistry=true：拉取注册表的本地副本
*	eureka.client.serviceUrl.defaultZone=http://localhost:8761/eureka/：Eureka服务的位置，包含客户端用于解析服务位置的Eureka服务的列表，该列表以逗号进行分隔。

可以使用Eureka的REST API来勘察注册表的内容。要查看服务的所有实例，可以以GET方法访问端点：
	
	http://<eureka service>:8761/eureka/apps/<APPID>
	
## 4.5 使用服务发现来查找服务

3个不同的Spring/Netflix客户端库，服务消费者可以使用它们来和Ribbon进行交互。从最低级别到最高级别，这些库包含了不同的与Ribbon进行交互的抽象层次。包括：

*	Spring DiscoveryCiient
*	启用了RestTemplate的Spring DiscoveryClient
*	Netflix Feign客户端

新路由：

	@RequestMapping(value="/{licenseId}/{clientType}", method=RequestMethod.GET}
	public License getLicensesWithClient(	
		@PathVariable("organizationId") String organizationId,
		@PathVariable("licenseId") String licenseId,
		@PathVariable("clientType") String clientType) {
			...
		}
	)
	
该路由上传递的clientType参数决定了我们将在代码示例中使用的客户端类型。可以在此路由上传递的具体类型包括：

*	Discovery——使用DiscoveryClient和标准的Spring RestTemplate类来调用组织服务
*	Rest——使用增强的Spring RestTemplate来调用基于Ribbon的服务
*	Feign——使用Netflix的Feign客户端来通过Ribbon调用服务

### 4.5.1 使用Spring DiscoveryClient查找服务实例

Spring DiscoveryClient提供了对Ribbon和Ribbon中缓存的注册服务的最低层次访问。使用DiscoveryClient，可以查询通过Ribbon注册的所有服务以及这些服务对应的URL。

@EnableDiscoveryClient注解是Spring Cloud的触发器，其作用是使应用程序能够使用DiscoveryClient和Ribbon库。

	@Autowired
	private DiscoveryClient discoveryClient;					// 用于和Ribbon交互的类
		
	discoveryClient.getInstances("xxx");						// 传入要查找的服务的关键字，以检索serverInstance对象的列表
	
	String serverUri = ...										// 构建目标URL
	
	restTemplate.exchange(serverUri, HttpMethod.GET,......);	// 使用标准的Spring REST模板类去调用服务
	
上述代码的问题：

*	没有利用Ribbon的客户端负载均衡
*	开发人员做了太多的工作

### 4.5.2 使用带有Ribbon功能的Spring RestTemplate调用服务

使用getRestTemplate()方法来创建支持Ribbon的Spring RestTemplate bean。

	@LoadBanlanced
	@Bean
	public RestTemplate getRestTemplate() {
		return new RestTemplate();
	}
	
@LoadBanced注解告诉Spring Cloud创建一个支持Ribbon的RestTemplate类

	restTemplate.exchange("http://{applicationid}/v1/...", httpMethod.GET, ...)
	
启用Ribbon的RestTemplate将解析传递给它的URL，并使用传递的内容作为服务器名称，该服务器名称作为从Ribbon查询服务实例的键。实际的服务位置和端口与开发人员完全抽象隔离。

此外，通过使用RestTemplate类，Ribbon将在所有服务实例之间轮询负载均衡所有请求。

### 4.5.3 使用Netflix Feign客户端调用服务

NetFlix的Feign客户端是Spring启用Ribbon的RestTemplate类的替代方案。Feign库采用不同的方法来调用REST服务，方法是让开发人员首先定义一个Java接口，然后使用Spring Cloud注解来标注接口，以映射Ribbon将要调用的基于Eureka的服务。Spring Cloud框架将动态生成一个代理类，用于调用目标REST服务。除了编写接口定义，开发人员不需要编写其他调用服务的代码。

@EnableFeign Clients：在代码中启用Feign客户端

	@FeignClient("xxxService")
	public interface XXXFeignClient {
		@RequestMapping(
			method = RequestMethod.GET,
			value = "/v1/...,
			consumes="application/json")
		XXX getXXX(
			@PathVariable("xxx") String xxx);
			
错误处理：通过Feign客户端，任何被调用的服务返回的HTTP状态码4xx-5xx都将映射为FeignException。

## 4.6 小结

*	服务发现模式用于抽象服务的物理位置。
*	诸如Eureka这样的服务发现引擎可以在不影响服务客户端的情况下，无缝地向环境中添加和从环境中移除服务实例。
*	通过进行服务调用的客户端中缓存服务的物理位置，客户端负载均衡可以提供额外的性能和弹性。
*	Eureka是Netflix项目，在与Spring Cloud一起使用时，很容易对Eureka进行建立和配置。
*	本章在Spring Cloud、Netflix Eureka和Netflix Ribbon中使用了3种不同的机制来调用服务。这些机制包括：
	*	使用Spring Cloud服务DiscoveryClient
	*	使用Spring Cloud和支持Ribbon的RestTemplate
	*	使用Spring Cloud和Netflix的Feign客户端
	
# 第5章 使用Spring Cloud和Netflix Hystrix的客户端弹性模式

本章主要内容：

*	实现断路器模式、后备模式和舱壁模式
*	使用断路器模式来保护微服务客户端资源
*	当远程服务失败时使用Hystrix
*	实施Hystrix的舱壁模式来隔离远程资源调用
*	调节Hystrix的断路器和舱壁的实现
*	定制Hystrix的并发策略

当服务运行缓慢时，检测到这个服务性能不佳并绕过它时非常困难的，这是因为以下几个原因：

1.	服务的降级可以以间歇性问题开始，并形成不可逆转的势头
2.	对远程服务的调用通常是同步的，并且不会缩短长时间运行的调用
3.	应用程序经常被设计为处理远程资源的彻底故障，而不是部分降级

性能不佳的远程服务所导致的潜在问题是，它们不仅难以检测，还会触发连锁效应，从而影响整个应用程序生态系统。如果没有适当的保护措施，一个性能不佳的服务可以迅速拖垮多个应用程序。

## 5.1 什么是客户端弹性模式

客户端弹性软件模式的重点是，在远程服务发生错误或表现不佳时保护远程资源（另一个微服务调用或数据库查询）的客户端免于崩溃。这些模式的目标时让客户端“快速失败”，而不消耗诸如数据库连接和线程池之类的宝贵资源，并且可以防止远程服务的问题向客户端的消费者进行“上游”传播。

有4种客户端弹性模式，它们分别是：

1.	客户端负载均衡（client load balance）模式：服务客户端缓存在服务发现期间检索到的微服务端点
2.	断路器（circuit breaker）模式；断路器模式确保服务客户端不会重复调用失败的服务
3.	后备（fallback）模式；当调用失败时，后备模式询问是否有可执行的替代方案
4.	舱壁（bulkhead）模式：舱壁模式隔离服务客户端上不同的服务调用，以确表现不佳的服务不会耗尽客户端的所有资源

### 5.1.1 客户端负载均衡模式

客户端负载均衡器位于服务客户端和服务消费者之间，所以负载均衡器可以检测服务实例是否抛出错误或表现不佳。如果客户端负载均衡器检测到问题，它可以从可用服务位置池中移除该服务实例，并防止将来的服务调用访问该服务实例。

这正是Netflix的Ribbon库提供的开箱即用的功能，而不需要额外的配置。

### 5.1.2 断路器模式

断路器模式时模仿电路断路器的客户端弹性模式。当远程服务被调用时，断路器将监视这个调用。如果调用时间太长，断路器将会介入并中断调用。此外，断路器将监视所有对远程资源的调用，如果对某一个远程资源的调用失败次数足够多，那么断路器实现就会出现并采取快速失败，阻止将来调用失败的远程资源。

### 5.1.3 后备模式

有了后备模式，当远程服务调用失败时，服务消费者将执行替代代码路径，并尝试通过其他方式执行操作，而不是生成一个异常。这通常涉及另一数据源查找数据或将用户的请求进行排队以供将来处理。用户的调用结果不会显示为提示问题的异常，但用户可能会被告知，他们的请求要在晚些时候被满足。

### 5.1.4 舱壁模式

通过使用舱壁模式，可以把远程资源的调用分到线程池中，并降低一个缓慢的远程资源调用拖垮整个应用程序的风险。线程池充当服务的“舱壁”。每个远程资源都是隔离的，并分配给线程池。如果一个服务响应缓慢，那么这种服务调用的线程池就会饱和并停止处理请求，而对其他服务的服务调用则不会变得饱和，因为它们被分配给了其他线程池。

## 5.2 为什么客户端弹性很重要

断路器在应用程序和远程服务之间充当中间人。

断路器会让少量的请求调用直达一个降级的服务，如果这些调用连续多次成功，断路器就会自动复位。

断路器模式为远程调用提供的关键能力：

1.	快速失败
2.	优雅地失败
3.	无缝回复

## 5.3 进入Hystrix

## 5.4 搭建许可服务器以使用Spring Cloud和Hystrix

pom.xml：

	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-hystrix</artifactId>
	</dependency>
	<dependency>
		<groupId>com.netflix.hystrix</groupId>
		<artifactId>hystrix-javanica</artifactId>
		<version>1.5.9</version>
	</dependency>
	
第一个依赖告诉Maven去拉取Spring Cloud Hystrix依赖项，第二个依赖标签将拉取核心Netflix Hystrix库。

@EnableCircuitBreaker：告诉Spring Cloud将要为服务使用Hystrix

## 5.5 使用Hystrix实现断路器

两大类别的Hystrix实现：

*	第一个类别：Hystrix包装的所有数据库调用
*	第二个类别：Hystrix包装的内部服务调用

Hystrix位于每个远程资源调用之间并保护客户端。

	@HystrixCommand
	public List<License> getLicensesByOrg(String organizationId) {
		return licenseRepository.findByOrganizationId(organizationId);
	}
	
@HystrixCommand注解会使用Hystrix断路器包装getLicenseByOrg()方法

使用@HystrixCommand注解，在任何时候调用getLicensesByOrg()方法时，Hystrix断路器都将包装这个调用。每当调用时间超过1000ms时，断路器将中断对getLicensesByOrg()方法的调用。

### 5.5.1 对组织微服务的调用超时

我们可以使用方法级注解使被标记的调用拥有断路器功能，其优点在于，无论是访问数据库还是调用微服务，它都是相同的注解。

在默认情况下，在指定不带属性的@HystrixCommand注解时，这个注解会将所有远程服务调用都放在同一线程池下。

### 5.5.2 定制断路器的超时时间

	@HystrixCommand(commandProperties = {@HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "12000")})
	
## 5.6 后备处理

断路器模式的一部分美妙之处在于，由于远程资源的消费者和资源本身之间存在“中间人”，因此开发人员有机会拦截服务故障，并选择替代方案。

	@HystrixCommand(fallbackMethod = "buildFallbackLicenseList")
	
fallbackMethod属性定义了类中的一个方法，如果来自Hystrix的调用失败，那么就会调用方法

## 5.7 实现舱壁模式

在基于微服务的应用程序中，开发人员通常需要调用多个微服务来完成特定的任务。在不使用舱壁模式的情况下，这些调用默认时使用同一批线程来执行调用的，这些线程是为了处理整个Java容器的请求而预留的。在存在大量请求的情况下，一个服务出现性能问题会导致Java容器的所有线程被刷爆并等待处理工作，同时堵塞新请求，最终导致Java容器崩溃。舱壁模式将远程资源调用隔离在它们自己的线程池中，以便可以控制单个表现不佳的服务，而不会使该容器崩溃。

Hystrix使用线程池来委派所有对远程服务的请求。在默认情况下，所有的Hystrix命令都将共享同一个线程池来处理请求。这个线程池将有10个线程来处理远程服务调用，而这些远程服务调用可以是任何东西，包括REST服务调用、数据库调用等。

在应用程序中访问少量的远程资源时，这种模型运行良好，并且各个服务的调用量分布相对均匀。问题是，如果某些服务具有比其他服务高得多得请求量或更长得完成时间，那么最终可能会导致Hystrix线程池中的线程耗尽，因为一个服务最终会占据默认线程池中的所有线程。

幸好，Hyxtrix提供了一种易于使用的机制，在不同的远程资源调用之间创建舱壁。

*	每个远程资源调用都放置在自己的线程池中。每个线程池都有可用于处理请求的最大线程数。
*	一个性能低下的服务只会影响同一线程池中的其他服务调用，从而限制了调用可能会造成的损害。

创建舱壁：

	@HystrixCommand(fallbackMethod = "buildFallbackLicenseList",
		threadPoolKey = "licenseByOrgThreadPool",
		threadPoolProperties = {
			@HystrixProperty(name = "coreSize", value = "30"),
			@HystrixProperty(name = "maxQueueSize", value = "10")}

threadPoolKey属性定义线程池的唯一名称；coreSize属性用于定义线程池中线程的最大数量；maxQueueSize用于定义一个位于线程池前的队列，它可以对传入的请求进行排队，该队列将控制在线程池中线程繁忙时允许堵塞的请求数。

有关maxQueueSize属性的两件事情：

1.	如果将其值设置为-1，则将使用Java SynchronousQueue来保存所有传入的请求。同步队列本质上会强制要求正在处理中的请求数量永远不能超过线程池中可用线程的数量。
2.	将maxQueueSize设置为大于1的值将导致Hystrix使用Java LinkedBlockingQueue。LinkedBlockingQueue的使用允许开发人员即使所有线程都在忙于处理请求，也能对请求进行排队。

要注意的第二件事时，maxQueueSize属性只能在线程池首次初始化时设置（例如，在应用程序启动时）。Hystrix允许通过使用queueSizeRejectionThreshold属性来动态更改队列的大小，但只有在maxQueueSize属性的值大于0时，才能设置此属性。

自定义线程池的适当大小是多少？Netflix推荐以下公式：

	服务在健康状态时每秒支撑的最大请求数 * 第99百分位延迟时间（以秒为单位） + 用于缓冲的少量额外线程

## 5.8 基础进阶——微调Hystrix

Hystrix在远程资源调用失败时使用的决策过程：

*	每当Hystrix命令遇到服务错误时，它将开始一个10s的计时器，用于检查服务调用失败的频率。这个10s窗口是可配置的。Hystrix做的第一件事情就是查看在10s内发生的调用数量。如果调用次数少于在这个窗口内需要发生的最小调用次数，那么即使有几个调用失败，Hystrix也不会采取行动。
*	在10s窗口内达到最少的远程资源调用次数时，Hystrix将开始查看整体故障的百分比。如果故障的总体百分比超过阈值，Hystrix将触发断路器，使将来几乎所有的调用都失败。
*	如果阈值超过错误阈值的百分比，Hystrix将“跳闸”断路器，防止更多的调用访问远程资源。如果远程调用失败的百分比未达到要求的阈值，并且10s窗口已过去，Hystrix将重置断路器的统计信息。
*	当Hystrix在一个远程调用上“跳闸”断路器时，它将尝试启动一个新的活动窗口。每隔5s（这个值时可配置的），Hystrix会让一个调用到达这个苦苦挣扎的服务。如果调用成功，Hystrix将重置断路器并重新开始让调用通过。如果调用失败，Hystrix将保持断路器断开，并在另一个5s里再尝试上述步骤。

基于此，开发人员可以使用5个属性来定制断路器的行为。@HystrixCommand注解通过commandPoolProperties属性公开了这5个属性。其中，threadPoolProperties属性用于设置Hystrix命令中使用的底层线程池的行为，而commandPoolProperties属性用于定制与Hystrix命令关联的断路器的行为。

*	circuitBreaker.requestVolumeThreshold用于控制Hystrix考虑将断路器跳闸之前，在10s之内必须发生的连续调用数量。
*	circuitBreaker.errorThresholdPercentage是在超过circuitBreaker.requestVolumeThreshold值之后在断路器跳闸之前必须达到的调用失败（由于超时、抛出异常或返回HTTP 500）百分比。
*	circuitBreaker.sleepWindowInMilliseconds是在断路器跳闸之后，Hystrix允许另一个调用通过以便查看服务是否回复健康之前Hystrix休眠时间。
*	metrics.rollingStats.timeInMilliseconds用于控制Hystrix用来监视服务调用问题窗口大小，其默认值为10000ms（即10s）。
*	metrics.rollingStats.numBuckets控制在定义的滚动窗口中收集统计信息的次数。

### 重新审视Hystrix配置

在配置Hystrix环境时，需要记住的关键点是，开发人员可以使用Hystrix的3个配置级别：

1.	整个应用程序级别的默认值
2.	类级别的默认值
3.	在类中定义的线程池级别

类级属性是通过一个名为@DefaultProperties的类级别注解设置的

用于创建和配置@HystrixCommand注解的所有配置值：

| 属性名称 | 默认值 | 描述 |
| -------- | ------ | ---- |
| fallbackMethod | None | 标识类中的方法，如果远程调用超时，将调用该方法。回掉方法必须与@HystrixCommand注解在同一个类中，并且必须具有与调用类相同的方法签名。如果值不存在，Hystrix会抛出异常 |
| threadPoolKey | None | 给予@HystrixCommand一个唯一的名称，并创建一个独立于默认线程池的线程池。如果没有定义任何值，则将使用默认的Hystrix线程池 |
| threadPoolProperties | None | 核心的Hystrix注解属性，用于配置线程池的行为 |
| coreSize | 10 | 设置线程池的大小 |
| maxQueueSize | -1 | 设置线程池前面的最大队列大小。如果设置为-1，则不使用队列，Hystrix将阻塞请求，直到有一个线程可用来处理 |
| circuitBreaker.requestVolumeThreshold | 20 | 设置Hystrix开始检查断路器是否跳闸之前滚动窗口中必须处理的最小请求数 |
| circuitBreaker.errorThresholdPercentage | 50 | 在断路器跳闸之前，滚动窗口内必须达到的故障百分比 |
| circuitBreaker.sleepWindowInMilliseconds | 5000 | 在断路器跳闸之后，Hystrix尝试进行服务调用之前将要等待的时间 |
| metricsRollingStats.timeInMilliseconds | 10000 | Hystrix收集和监控服务调用的统计信息的滚动窗口（以毫秒为单位） |
| metricsRollingStats.numBuckets | 10 | Hystrix在一个监控窗口中维护的度量桶的数量。监视窗口内的桶数越多，Hystrix在窗口内监控故障的时间越低 |

## 5.9 线程上下文和Hystrix

当一个@HystrixCommand被执行时，它可以使用两种不同的隔离策略——THREAD（线程）和SEMAPHORE（信号量）来运行。在默认情况下，Hystrix以THREAD隔离策略运行。用于保护调用的每个Hystrix命令都在一个单独的线程池中运行，该线程池不与父线程共享它的上下文。这意味着Hystrix可以在它的控制下中断线程的执行，而不必担心中断与执行原始调用的父线程相关的其他活动。

通过基于SEMAPHORE的隔离，Hystrix管理由@HystrixCommand注解保护的分布式调用，而不需要启动一个新线程，并且如果调用超时，就会中断父线程。在同步容器服务器环境（Tomcat）中，中断父线程将导致抛出开发人员无法捕获的异常。这可能会给编写代码的开发人员带来意想不到的后果，因为他们无法捕获抛出的异常或执行任何资源清理或错误处理。

要控制命令池的隔离设置，开发人员可以在自己的@HystrixCommand注解上设置commandProperties属性。如：

	@HystrixCommand(
		commandProperties = {
			@HystrixProperty(name = "execution.islation.strategy", value = "SEMAPHORE")})
			
### 5.9.1 ThreadLocal与Hystrix

幸运的是，Hystrix和Spring Cloud提供了一种机制，可以将父线程的上下文传播到由Hystrix线程池管理的线程。这种机制被称为HystrixConcurrencyStrategy。

### 5.9.2 HystrixConcurrencyStrategy实战

需要执行以下3个操作：

1.	定义自定义的Hystrix并发策略类
2.	定义一个Callable类，将UserContext注入Hystrix命令中
3.	配置Spring Cloud以使用自定义Hystrix并发策略

#### 1. 自定义Hystrix并发策略类

	public class ThreadLocalAwareStrategy extends HystrixConcurrencyStrategy {
		
		private HystrixConcurrencyStrategy existingConcurrencyStrategy;
		
		...
		
		@Override
		public <T> Callable<T> wrapCallable(Callable<T> callable) {
			
			return existingConcurrencyStrategy != null ?
				existingConcurrencyStrategy.wrapCallable(new DelegatingUserContextCallable<T>(
					callbale, UserContextHolder.getContext())) :
				super.wrapCallable(
					new DelegatingUserContextCallable<T>(
						callable, UserContextHolder.getContext()));
		}
	}
	
首先，因为Spring Cloud已经定义了一个HystrixConcurrencyStrategy，所以所有可能被覆盖的方法都需要检查现有的并发策略是否存在，然后或调用现有的并发策略的方法或调用基类的Hystrix并发策略方法。

第二件事是wrapCallable()方法。在此方法中，我们传递了Callable的实现DelegatingUserContextCallable，用来将UserContext从执行用户REST服务调用的父线程，设置为保护正在进行工作的方法的Hystrix命令线程。

#### 2. 定义一个Java Callable类，将UserContext注入Hystrix命令中

	public final class DelegatingUserContextCallable<V>
		implements Callable<V> {
	
		private final Callable<V> delegate;
		private UserContext originalUserContext;
		
		public DelegatingUserContextCallable(
			Callable<V> delegate, UserContext userContext) {
			this.delegate = delegate;
			this.originalUserContext = userContext;
		}
		
		public V call() throws Exception {
			UserContextHolder.setContext(originalUserContext);
			try {
				return delegate.call();
			} finally {
				this.originalUserContext = null;
			}
		}
		
		public static <V> Callable<V> create(Callable<V> delegate, UserContext userContext) {
			return new DelegatingUserContextCallable<V>(delegate, userContext);
		}
	}
	
当调用Hystrix保护的方法时，Hystrix和Spring Cloud将实例化DelegatingUserContextCallable类的一个实例，传入一个通常由Hystrix命令池管理的线程调用的Callable类。

除了委托的Callable类之外，Spring Cloud也将UserContext对象从发起调用的父线程传递出去。这两个值在创建DelegatingUserContextCallable实例时设置，实际的操作将发生在类的call()方法中。

#### 3. 配置Spring Cloud以使用自定义Hystrix并发策略

## 5.10 小结

*	在涉及高分布式应用程序（如基于微服务的应用程序）时，必须考虑客户端弹性。
*	服务的彻底故障（如服务器崩溃）是很容易检测和处理的。
*	一个性能不佳的服务可能会引起资源耗尽的连锁效应，因为调用客户端中的线程被阻塞，以等待服务完成。
*	3种核心客户端弹性模式分别是断路器模式、后备模式和舱壁模式。
*	断路器模式试图杀死运行缓慢和降级的系统调用，这样调用就会快速失败，并防止资源耗尽。
*	后备模式允许开发人员在远程服务调用失败或断路器跳闸的情况下，定义替代代码路径。
*	舱壁模式通过将对远程服务的调用隔离到它们自己的线程池中，使远程资源调用彼此分离。就算一组服务调用失败，这些失败也不会导致应用程序容器中的所有资源耗尽。
*	Spring Cloud和Netflix Hystrix库提供断路器模式、后备模式和舱壁模式的实现。
*	Hystrix库是高度可配置的，可以在全局、类和线程池级别设置。
*	Hystrix支持两种隔离模型，即THREAD和SEMPHORE。
*	Hystrix默认隔离模型THREAD完全隔离Hystrix保护的调用，但不会将父线程的上下文传播到Hystrix管理的线程。
*	Hystrix的另一种隔离模型SEMAPHORE不使用单独的线程进行Hystrix调用。虽然这更有效率，但如果Hystrix中断了调用，它会让服务变得不可预测。
*	Hystrix允许通过自定义HystrixConcurrencyStrategy实现，将父线程上下文注入Hystrix管理的线程中。

# 第6章 使用Spring Cloud和Zuul进行服务路由

本章主要内容：

*	结合微服务使用服务网关
*	使用Spring Cloud和Netflix Zuul实现服务网关
*	在Zuul中映射微服务路由
*	构建过滤器以使用关联ID并进行跟踪
*	使用Zuul进行动态路由

在像微服务架构这样的分布式架构中，需要确保跨多个服务调用的关键行为的正常运行，如安全、日志记录和用户跟踪。要实现此功能，开发人员需要在所有服务中始终如一地强制这些特征，而不需要每个开发团队都构建自己的解决方案。使用公共库和框架的问题：

*	第一，在构建的每个服务中很难始终实现这些功能。
*	第二，正确地实现这些功能是一个挑战。
*	第三，这会在所有服务中创建一个顽固的依赖。

为了解决这个问题，需要将这些横切关注点抽象成一个独立且作为应用程序中所有微服务调用的过滤器和路由器的服务。这种横切关注点被称为服务网关（service gateway）。服务客户端不再直接调用服务。取而代之的是，服务网关作为单个策略执行点（Policy Enforcement Point, PEP），所有调用都通过服务网关进行路由，然后被路由到最终目的地。

使用Spring Cloud和Zuul来完成以下操作：

*	将所有服务调用放在一个URL后面，并使用服务发现将这些调用映射到实际的服务实例。
*	将关联ID注入流经服务网关的每个服务调用中。
*	在从客户端发回的HTTP响应中注入关联ID。
*	构建一个动态路由机制，将各个具体的组织路由到服务实例端点，该端点与其他人使用的服务实例端点不同。