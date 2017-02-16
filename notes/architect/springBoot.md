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
