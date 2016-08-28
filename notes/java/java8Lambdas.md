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

| 接口 | 参数 | 返回类型 | 示例 |
| ---- | ---- | ---- | ---- |
| Predicate&lt;T&gt; | T | boolean | 这张唱片已经发行了吗 | 
| Consumer&lt;T&gt; | T | void | 输出一个值 |
| Function&lt;T, R&gt; | T | R | 获得Artist对象的名字 |
| Supplier&lt;T&gt; | None | T | 工厂方法 |
| UnaryOperator&lt;T&gt; | T | T | 逻辑非(!) |
| BinaryOperator&lt;T&gt; | (T, T) | T | 求两个数的乘积(*) |

## 2.5 类型推断

    Predicate<Integer> alLeast5 = x -> x > 5;
    BinaryOperator<Long> addLongs = (x, y) -> x + y;

没有泛型，代码则通不过编译

    BinaryOperator add = (x, y) -> x + y;

## 2.6 要点回顾

*   Lambda表达式是一个匿名方法，将行为像数据一样进行传递。
*   Lambda表达式的常见结构：BinaryOperator<Integer> add = (x, y) -> x + y。
*   函数接口指仅具有单个抽象方法的接口，用来表示Lambda表达式的类型。

## 2.7 练习

# 第3章 流

流使程序员得以站在更高的抽象层次上对集合进行操作。

## 3.1 从外部迭代到内部迭代

    long count = allArtists.stream()
            .filter(artist -> artist.isFrom("Londom"))
            .count();

stream是用函数式编程方式在集合类上进行复杂操作的工具

## 3.2 实现机制

*   像filter这样只描述Stream，最终不产生新集合的方法叫作**惰性求值方法**，惰性求值的返回值是Stream；
*   像count这样最终会从Stream产生值的方法叫作**及早求值方法**，及早求值返回值是另一个值或为空。 

## 3.3 常用的流操作

### 3.3.1 collect(toList())

collect(toList())方法由Stream里的值生成一个列表，是一个及早求值操作。

    List<String> collected = Stream.of("a", "b", "c")
            .collect(Collectors.toList());

### 3.3.2 map

如果有一个函数可以将一种类型的值转换成另外一种类型，map操作就可以使用该函数，将一个流中的值转换成一个新的流。

    List<String> collected = Stream.of("a", "b", "hello")
            .map(string -> string.toUpperCase())
            .collect(toList());

### 3.3.3 filter

遍历数据并检查其中的元素时，可尝试使用Stream中提供的新方法filter

    List<String> beginningWithNumbers = Stream.of("a", "1abc", "abc1")
            .filter(value -> isDigit(value.charAt(0)))
            .collect(toList());

### 3.3.4 flatMap

flatMap方法可用Stream替换值，然后将多个Stream连接成一个Stream。

    List<Integer> together = Stream.of(asList(1, 2), asList(3, 4))
            .flatMap(numbers -> numbers.stream())
            .collect(toList());

### 3.3.5 max和min

