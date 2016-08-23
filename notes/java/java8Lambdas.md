---
layout: post
notes: true
subtitle: "Java 8 Lambdas: Functional Programming for the Masses"
comments: false
author: "[英] Richard Warburton（王群锋 译）"
date: 2016-08-23 12:00:00

---


![](/img/notes/java/java8Lambdas/java8_lambdas.jpeg)

*   目录
{:toc }

# 第1章 简介

## 1.1 为什么需要再次修改Java

面向对象编程是对数据进行抽象，而函数式编程是对行为进行抽象。

## 1.2 什么是函数式编程

核心：在思考问题时，使用不可变值和函数，函数对一个值进行处理，映射成另一个值。

## 1.3 示例

# 第2章 Lambda表达式

Java 8的最大变化是引入了Lambda表达式——一种紧凑的、传递行为的方式。

## 2.1 第一个Lambda表达式

    button.addActionListener(event->System.out.println("button clicked"))