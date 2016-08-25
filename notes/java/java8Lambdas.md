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

尽管与之前相比，Lambda表达式中的参数需要的样板代码很少，但是Java 8仍然是一种静态类型语言。为了增加可读性并迁就我们的习惯，，声明参数时也可以包括类型信息，而且有时编译器不一定能根据上下文推断出参数的类型！

## 2.2 如何辨别Lambda表达式

Lambda表达式的集中变体：

    Runnable noArguments = () -> System.out.println("Hello World");

上述所示的Lambda表达式不包含参数，使用空括号()表示没有参数。该Lambda表达式实现了Runnable接口，该接口也只有一个run方法，没有参数，且返回类型为void。

    ActionListener oneArgument = event -> System.out.println("button clicked");

上述所示的Lambda表达式包含且只包含一个参数，可省略参数的括号。

    Runnable multiStatement = () -> {
        System.out.print("Hello");
        System.out.println(" World");
    }

如上，表达式的主体不仅可以是一个表达式，而且也可以是一段代码块。

    BinaryOperator<Long> add = (x, y) -> x + y;

如上，Lambda表达式也可以表示包含多个参数的方法，这时就有必要思考怎样去阅读该Lambda表达式。这行代码并不是将两个数字相加，而是创建了一个函数，用来计算两个数字相加的结果。变量add的类型是BinaryOperator<Long>，它不是两个数字的和，而是将两个数字相加的那行代码。

    BinaryOperator<Long> addExplicit = (Long x, Long y) -> x + y;

如上，有时最好可以显式声明参数类型。

目标类型是指Lambda表达式所在上下文环境的类型。

Lambda表达式的类型依赖于上下文环境，是由编译器推断出来的。

## 2.3 引用值，而不是变量

虽然无需将变量声明为final，但在Lambda表达式中，也无法用作非终态变量，如果坚持用作非终态变量，编译器就会报错。

## 2.4 函数接口

函数接口是只有一个抽象方法的接口，用作Lambda表达式的类型。
