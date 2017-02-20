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