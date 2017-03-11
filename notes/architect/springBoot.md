---
layout: post
notes: true
subtitle: "Java EE开发的颠覆者——Spring Boot实战"
comments: false
author: "汪云飞"
date: 2017-02-16 00:00:00

---

![](/img/notes/architect/springBoot/spring_boot.jpg)

*   目录
{:toc }

# 第一章 Spring 基础

## 1.1 Spring 概述

### 1.1.1 Spring 的简史

*	第一阶段：xml配置
*	第二阶段：注解配置
*	第三阶段：Java配置

### 1.1.2 Spring 概述

Spring 提供了一个IoC容器用来初始化对象，解决对象间的依赖管理和对象的使用。

#### 1. Spring的模块

Spring 是模块化的，这意味着你可以只使用你需要的Spring的模块。

![](/img/notes/architect/springBoot/spring_module.jpg)

##### （1）核心容器（Core-Container）

*	Spring-Core：核心工具类，Spring 其他模块大量使用 Spring-Core
*	Spring-Beans：Spring 定义Bean的支持
*	Spring-Context：运行时 Spring 容器
*	Spring-Context-Support：Spring 容器对第三方包的集成支持
*	Spring-Expression：使用表达式语言在运行时查询和操作对象

##### （2）AOP

*	Spring-AOP：基于代理的 AOP 支持
*	Spring-Aspects：基于AspectJ的 AOP 支持

##### （3）消息（Messaging）

*	Spring-Messaging：对消息架构和协议的支持

##### （4）Web

*	Spring-Web：提供基础的 Web 集成的功能，在 Web 项目中提供 Spring 的容器
*	Spring-Webmvc：提供基于 Servlet 的 Spring MVC
*	Spring-WebSocket：提供 WebSocket 功能
*	Spring-Webmvc-Portlet：提供 Portlet 环境支持

##### （5）数据访问/集成（Data Access/Integration）

*	Spring-JDBC：提供以 JDBC 访问数据库的支持
*	Spring-TX：提供编程式和声明式的事务支持
*	Spring-ORM：提供对对象/关系映射技术的支持
*	Spring-OXM：提供对对象/xml映射技术的支持
*	Spring-JMS：提供对JMS的支持

#### 2. Spring的生态

*	Spring Boot：使用默认开发配置来实现快速开发
*	Spring XD：用来简化大数据应用开发
*	Spring Cloud：为分布式系统开发提供工具集
*	Spring Data：对主流的关系型和 NoSQL 数据库的支持
*	Spring Integration：通过消息机制对企业集成模式（EIP）的支持
*	Spring Batch：简化及优化大量数据的批处理操作
*	Spring Security：通过认证和授权保护应用
*	Spring HATEOAS：基于 HATEOAS 原则简化 REST 服务开发
*	Spring Social：与社交网络API（如Facebook、新浪微博等）的集成
*	Spring AMQP：对基于 AMQP 的消息的支持
*	Spring Mobile：提供对手机设备检测的功能，给不同的设备返回不同的页面的支持
*	Spring for Android：主要提供在 Android 上消费 RESTful API 的功能
*	Spring Web Flow：基于 Spring MVC 提供基于向导流程式的 Web 应用开发
*	Spring Web Services：提供了基于协议有限的 SOAP/Web 服务
*	Spring LDAP：简化使用 LDAP 开发
*	Spring Session：提供一个 API 及实现来管理用户会话信息

## 1.2 Spring 项目快速搭建

### 1.2.1 Maven 简介

Apache Maven 是一个软件项目管理工具。基于项目对象模型（Project Object Model, POM）的概念，Maven可用来管理项目的依赖、编译、文档等信息。

### 1.2.2 Maven 安装

### 1.2.3 Maven 的 pom.xml

Maven 是基于项目对象模型的概念运作的，所以 Maven 的项目都有一个 pom.xml 用来管理项目的依赖以及项目的编译等功能。

### 1.2.4 Spring 项目的搭建

## 1.3 Spring 基础配置

Spring 框架本身有四大原则：

1.	使用POJO进行轻量级和最小侵入式开发。
2.	通过依赖注入和基于接口编程实现松耦合。
3. 	通过AOP和默认习惯进行声明式编程。
4.	使用AOP和模板（template）减少模式化代码。

### 1.3.1 依赖注入

#### 1. 点睛

控制翻转（Inversion of Control-IOC）和依赖注入（dependency injection-DI）在 Spring 环境下是等同的概念，控制翻转是通过依赖注入实现的。所谓依赖注入指的是容器负责创建对象和维护对象间的依赖关系，而不是通过对象本身负责自己的创建和解决自己的依赖。

依赖注入的主要目的是为了解耦，体现了一种“组合”的理念。

Sprint IoC 容器（ApplicationContext）负责创建 Bean，并通过容器将功能类Bean注入到你需要的 Bean 中。Spring 提供使用 xml、注解、Java 配置、groovy 配置实现 Bean 的创建和注入。

声明 Bean 的注解：

*	@Component：组件，没有明确的角色
*	@Service：在业务逻辑层（service层）使用
*	@Repository：在数据访问层（dao层）使用
*	@Controller：在展现层（MVC->Spring MVC）使用

注入 Bean 的注解，一般情况下通用：

*	@Autowired：Spring 提供的注解
*	@Inject：JSR-330 提供的注解
*	@Resource：JSR-250 提供的注解

#### 2. 示例

配置类：

	@Configuration
	@ComponentScan("com.wisely.highlight_spring4.ch1.di")
	public class DiConfig {
	}
	
*	@Configuration 声明当前类是一个配置类
*	@ComponentScan：自动扫描包名下所有使用@Service、@Component、@Repository和@Controller 的类，并注册为 Bean。

### 1.3.2 Java配置

#### 1. 点睛

Java 配置是 Spring 4.x 推荐的配置方式，可以完全替代 xml 配置；Java 配置也是 Spring Boot 推荐的配置方式。

Java 配置是通过 @Configuration 和 @Bean 来实现的：

*	@Configuration 声明当前类是一个配置类，相当于一个 Spring 配置的 xml 文件。
*	@Bean 注解在方法上，声明当前方法的返回值为一个 Bean。

原则：

*	全局配置使用 Java 配置（如数据库相关配置、MVC 相关配置）
*	业务 Bean 的配置实用注解配置（@Service、@Component、@Repository、@Controller）

#### 2. 示例

	@Configuration
	public class JavaConfig {
		
		@Bean 
		public FunctionService functionService() {
			return new FunctionService();
		}
		
		@Bean 
		public UseFunctionService useFunctionService() {
			UseFunctionService useFunctionService = new UseFunctionService();
			useFunctionService.setFunctionService(functionService());
			return useFunctionService;
		}
		
	//	@Bean
	//	public useFunctionService useFunctionService(FunctionService functionService) {
	//		UseFunctionService useFunctionService = new UseFunctionService();
	//		useFunctionService.setFunctionService(functionService);
	//		return useFunctionService;
		}
	}
	
在 Spring 容器中，只要容器中存在某个 Bean，就可以在另外一个 Bean 的声明方法的参数中注入。

### 1.3.3 AOP

#### 1. 点睛

AOP：面向切面编程，相对于 OOP 面向对象编程。

Spring 的 AOP 的存在目的是为了解耦。AOP 可以让一组类共享相同的行为。

Spring 支持 AspectJ 的注解式切面编程。

1.	使用 @Aspect 声明是一个切面
2.	使用 @After、@Before、@Around 定义建言（advice），可直接将拦截规则（切点）作为参数。
3.	其中 @After、@Before、@Around 参数的拦截规则为切点（PorintCut），为了使切点复用，可使用 @PointCut 专门定义拦截规则，然后在 @After、@Before、@Around的参数中调用。
4.	其中符合条件的每一个被拦截处为连接点（JointPoint）。

#### 2. 示例

配置类：

	@Configuration
	@ComponentScan("com.wisely.highlight_spring4.ch1.aop")
	@EnableAspectJAutoProxy
	public class AopConfig {
	}
	
使用 @EnableAspectJAutoProxy 注解开启 Spring 对 AspectJ 的支持。

# 第2章 Spring 常用配置

## 2.1 Bean 的 Scope

### 2.1.1 点睛

Scope 描述的是 Spring 容器如何新建 Bean 的实例的。Spring 的 Scope 有以下几种，通过 @Scope 注解来实现。

1.	Singleton：一个 Spring 容器中只有一个 Bean 的实例，此为 Spring 的默认配置，全容器共享一个实例。
2.	Prototype：每次调用新建一个 Bean 的实例。
3.	Request：Web 项目中，给每一个 http request 新建一个 Bean 实例。
4.	Session：Web 项目中，给每一个 http session 新建一个 Bean 实例。
5.	GlobalSession：这个只在 protal 应用中有用，给每一个 global http session 新建一个 Bean 实例。

另外，在 Spring Batch 中还有一个 Scope 是使用@StepScope。

### 2.1.2 示例

	@Service
	@Scope("prototype")
	public class DemoPrototypeService {
	}
	
## 2.2 Spring EL 和资源调用

### 2.2.1 点睛

Spring EL-Spring 表达式语言，支持在 xml 和注解中使用表达式，类似于 JSP 的 EL 表达式语言。

Spring 主要在注解 @Value 的参数中使用表达式。

### 2.2.2 示例

注入普通字符串：

	@Service
	public class DemoService {
		@Value("其他类的属性")
		private String another;
	}
	
配置类：

	@Configuration
	@ComponentScan("com.wisely.highlight_spring.ch2.el")
	@PropertySource("classpath:com/wisely/highlight_spring4/ch2/el/test.properties")
	public class ElConfig {
		
		// 注入普通字符串
		@Value("I Love You")
		private String normal;
		
		// 注入操作系统属性
		@Value("#{systemProperties['os.name']}")
		private String osName;
		
		// 注入表达式结果
		@Value("#{ T(java.lang.Math).random() * 100.0 }")
		private double randomNumber;
		
		// 注入其他Bean属性
		@Value("#{demoService.another}")
		private String fromAnother;
		
		// 注入文件资源
		@Value("classpath:com/wisely/highlight_spring4/ch2/el/test.txt")
		private Resource testFile;
		
		// 注入网址资源
		@Value("http://www.baidu.com")
		private Resource testUrl;
		
		// 注入配置文件
		@Value("${book.name}")
		private String bookName;
	}
	
## 2.3 Bean 的初始化和销毁

### 2.3.1 点睛

1.	Java配置方式：使用 @Bean 的 initMethod 和 destroyMethod （相当于 xml 配置的 init-method 和 destroy-method）
2.	注解方式：利用 JSR-250 的 @PostConstruct 和 @PreDestory。

### 2.3.2 演示

(1). 增加 JSR250 支持：

	<dependency>
		<groupId>javax.annotation</groupId>
		<artifactId>jsr250-api</artifactId>
		<version>1.0</version>
	</dependency>
	
(2). 使用 @Bean 形式的Bean。

	public class BeanWayService {
		public void init() {
			System.out.println("@Bean-init-method");
		}
		public BeanWayService() {
			super();
			System.out.println("初始化构造函数-BeanWayService");
		}
		public void destroy() {
			System.out.println("@Bean-destroy-method");
		}
	}
	
(3). 使用 JSR250 形式的Bean。

	public class BeanWayService {
		@PostConstruct
		public void init() {
			System.out.println("@Bean-init-method");
		}
		public BeanWayService() {
			super();
			System.out.println("初始化构造函数-BeanWayService");
		}
		@PreDestroy
		public void destroy() {
			System.out.println("@Bean-destroy-method");
		}
	}
	
解释：

1.	@PostConstruct，在构造函数执行完之后执行
2.	@PreDestroy，在 Bean 销毁之前执行

(4). 配置类

	@Configuration
	@ComponentScan("com.wisely.highlight_spring4.ch2,prepost")
	public class PrePostConfig {
		@Bean(initMethod="init", destroyMethod="destroy")
		BeanWayService beanWayService() {
			return new BeanWayService();
		}
		
		@Bean
		JSR250WayService jsr250WayService() {
			return new JSR250WayService();
		}
	}
	
## 3.4 条件注解@Conditional

### 3.4.1 点睛

Spring 4 提供了一个更通用的基于条件的Bean的创建，即是用@Conditional注解。

@Conditional根据某一特定条件创建一个特定的Bean。

### 3.4.2 示例

#### 1. 判断条件定义

	public class WindowsCondition implements Condition {
	
		public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
			return context.getEnvironment().getProperty("os.name").contains("Windows");
		}
	}
	
	public class LinuxCondition implements Condition {
	
		public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
			return context.getEnvironment().getProperty("os.name").contains("Linux");
		}
	}
	
#### 2. 不同系统下Bean的类

	public interface ListService {
		public String showListCmd();
	}
	
	public class WindowsListService implements ListService {
		
		@Override
		public String showListCmd() {
			return "dir";
		}
	}
	
	public class LinuxListService implements ListService {
		
		@Override
		public String showListCmd() {
			return "ls";
		}
	}
	
#### 3. 配置类

	@Configuration
	public class ConditionConfig {
		
		@Bean
		@Conditional(WindowsCondition.class)
		public ListService windowsListService() {
			return new WindowsListService();
		}
		
		@Bean
		@Conditional(LinuxCondition.class)
		public ListService linuxListService() {
			return new LinuxListService();
		}
	}

	
# 第5章 Spring Boot 基础

## 5.1 Spring Boot 概述

### 5.1.1 什么是 Spring Boot

Spring Boot 使用“习惯优于配置”（项目中存在大量的配置，此外还内置一个习惯性的配置，让你无须手动进行配置）的理念让你的项目快速运行起来，使用Spring Boot 很容易创建一个独立运行（运行jar，内嵌Servlet容器）、准生产级别的基于 Spring 框架的项目，使用 Spring Boot 你可以不用或者只需要很少的 Spring 配置。

### 5.1.2 Spring Boot 核心功能

#### 1. 独立运行的 Spring 项目

Spring Boot可以以jar包的形式独立运行，运行一个 Spring Boot 项目只需要通过 java -jar xx.jar 来运行。

#### 2. 内嵌 Servlet 容器

Spring Boot 可选择内嵌 Tomcat、Jetty 或者 Undertow，这样我们无须以 war 包形式部署项目。

#### 3. 提供 starter 简化 Maven 配置

Spring 提供了一系列的 starter pom 来简化 Maven 的依赖加载。

#### 4. 自动配置 Spring

Spring Boot 会根据在类路径中的jar包、类，为jar包里的类自动配置Bean，这样会极大地减少我们要使用的配置。

#### 5. 准生产的应用监控

Spring Boot 提供基于 http、ssh、telnet 对运行时的项目进行监控。

#### 6. 无代码生成和xml配置

Spring Boot 的神奇的不是借助于代码生成来实现的，而是通过条件注解来实现的，Spring Boot 不需要任何xml配置即可实现 Spring 的所有配置。

### 5.1.3 Spring Boot 的优缺点

优点：

1.	快速构建项目；
2.	对主流开发框架的无配置集成；
3.	项目可独立运行，无须外部依赖Servlet容器；
4.	提供运行时的应用监控；
5.	极大地提高了开发、部署效率；
6.	与云计算的天然集成。

缺点：

1.	书籍文档较少且不够深入
2.	如果你不认同Spring框架...

### 5.1.4 关于本书的 Spring Boot 版本

1.3.0

## 5.2 Spring Boot 快速搭建

### 5.2.1 http://start.spring.io

### 5.2.2 Spring Tool Suite

### 5.2.3 IntelliJ IDEA

### 5.2.4 Spring Boot CLI

Spring Boot CLI 是 Spring Boot 提供的控制台命令工具

### 5.2.5 Maven 手工构建

### 5.2.6 简单演示

@SpringBootApplication：Spring Boot 项目的核心注解，主要目的是开启自动配置。

通过Maven命令运行项目：

	mvn spring-boot:run

# 第6章 Spring Boot 核心

## 6.1 基本配置

### 6.1.1 入口类和SpringBootApplication

Spring Boot 通常有一个名为 *Application的入口类，入口类里有一个 main 方法，这个 main 方法其实就是一个标准的 Java 应用的入口方法。在 main 方法中使用 SpringApplication.run(*Application.class, args)，启动 Spring Boot 应用项目。

@SpringBootApplication 是 Spring Boot 的核心注解，它是一个组合注解，源码如下：

	@Target(ElementType.TYPE)
	@Retention(RetentionPolicy.RUNTIME)
	@Documented
	@Inherited
	@Configuration
	@EnableAutoConfiguration
	@ComponetScan
	public @interface SpringBootApplication {
		Class<?> exclude() default {};
		String[] excludeName() default {};
	}
	
@SpringBootApplication注解主要组合了@Configuration、@EnableAutoConfiguration、@ComponetScan；若不使用@SpringBootApplication注解，则可以在入口类上直接使用@Configuration、@EnableAutoConfiguration、@ComponentScan。

其中，@EnableAutoConfiguration让Spring Boot根据类路径中的jar包依赖为当前项目进行自动配置。

*	如添加了spring-boot-starter-web依赖，会自动添加 Tomcat 和 Spring MVC 的依赖，那么 Spring Boot 会对 Tomcat 和 Spring MVC 进行自动配置。
*	如添加了spring-boot-starter-data-jpa依赖，Spring Boot 会自动进行JPA相关的配置。

Spring Boot 会自动扫描@SpringBootApplication 所在类的同级包以及下极包里的Bean。建议入口类放置的位置在 groupId+architectID 组合的包名下。

### 6.1.2 关闭特定的自动配置

	@SpringBootApplication(exclude = {XXX.class})
	
### 6.1.3 定制Banner

#### 1. 修改Banner

在 src/main/resources 下新建一个banner.txt，写入Banner相关的字符，再启动程序即可。

#### 2. 关闭banner

main 里的内容修改为：

	new SpringApplicationBuilder(XXX.class)
		.showBanner(false)
		.run(args);
		
### 6.1.4 Spring Boot 的配置文件

Spring Boot 使用一个全局的配置文件 application.properties 或 application.yml，放置在 src/main/resources 目录或者类路径的/config 下。

Spring Boot 不仅支持常规的 properties 配置文件，还支持 yaml 语言的配置文件。yaml 是以数据为中心的语言，在配置数据的时候具有面向对象的特征。

application.properties：

	server.port = 9000
	server.context-path = /helloboot
	
application.yml：

	server:
		port : 9090
		contextPath : /helloboot
		
在Spring Boot中，context-path、contextPath或者CONTENT_PATH形式其实是通用的。

### 6.1.5 starter pom

Spring Boot为我们提供了简化企业级开发绝大多数场景的starter pom，只要使用了应用场景所需要的starter pom，相关的技术配置将会消除，就可以得到Spring Boot为我们提供的自动配置的Bean。

1.	官方start pom
2.	第三方starter pom

### 6.1.6 使用xml配置

	@ImportResource({"classpath:some-content.xml","classpath:another-context.xml"})
	
## 6.2 外部配置

Spring Boot 允许使用properties文件、yaml文件或者命令行参数作为外部配置。

### 6.2.1 命令行参数配置

	java -jar xx.jar
	
可以通过以下命令修改Tomcat端口号：

	java -jar xx.jar --server.port=9090
	
### 6.2.2 常规属性配置

application.properties：

	book.author=wangyunfei
	book.name=spring boot
	
修改入口类：

	public class Application {
		
		@Value("${book.author}")
		private String bookAuthor;
		
		@Value("${book.name}")
		private String bookName;
		
		...
	}
	
### 6.2.3 类型安全的配置（基于properties）

application.properties：

	author.name=wyf
	author.age=32
	
类型安全的Bean：

	@Component
	@ConfigurationProperties(prefix = "author")
	public class AuthorSettings {
		private String name;
		private Long age;
		
		// Setter and Getter
		...
	}
	
通过@ConfigurationProperties加载properties文件内的配置，通过prefix属性指定properties的配置的前缀，通过locations指定properties文件的位置。

检验代码：

	public class Application {
		
		@Autowired
		private AuthorSettings authorSettings;
		
		...
	}
	
可以用@Autowired直接注入该配置

## 6.3 日志配置

Spring Boot 支持Java Util Logging、Log4J、Log4J2和Logback作为日志框架，无论使用哪种日志框架，Spring Boot已为当前使用日志框架的控制台输出及文件输出做好了配置。

默认情况下，Spring Boot 使用Logback作为日志框架。

配置日志级别：

	logging.file=D:/mylog/log.log
	
配置日志文件，格式为logging.level.包名=级别：

	logging.level.org.springframework.web=DEBUG