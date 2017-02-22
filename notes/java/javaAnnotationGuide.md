---
layout: post
notes: true
subtitle: "【技术博客】Java 注解指导手册-终极向导"
comments: false
author: "Dani Buiza（Toien Liu译）"
date: 2016-10-22 12:00:00

---


原文：[https://www.javacodegeeks.com/2014/11/java-annotations-tutorial.html](https://www.javacodegeeks.com/2014/11/java-annotations-tutorial.html)
译文：[http://ifeve.com/java-annotations-tutorial/](http://ifeve.com/java-annotations-tutorial/)

*   目录
{:toc }

# 1. 什么是注解？

注解早在J2SE1.5就被引入到Java中，主要提供一种机制，这种机制允许程序员在编写代码的同时可以直接编写元数据。

包含注解的设计和开发的Java规范主要有以下两篇：

*	[JSR 175 A metadata facility for the Java programming Language](https://www.jcp.org/aboutJava/communityprocess/final/jsr175/index.html)
*	[JSR 250 Common Annotations for the Java Platform](https://jcp.org/en/jsr/detail?id=250)

# 2. 介绍

解释何为注解的最佳方式就是元数据这个词：描述数据自身的数据。注解就是代码的元数据，他们包含了代码自身的信息。

注解可以被用在包，类，方法，变量，参数上。自Java8起，有一种注解几乎可以被放在代码的任何位置，叫做类型注解。

被注解的代码并不会直接被注解影响。这只会向第三系统提供关于自己的信息以用于不同的需求。

注解会被编译至class文件中，而且会在运行时被处理程序提取出来用于业务逻辑。当然，创建在运行时不可用的注解也是可能的，甚至可以创建只在源文件中可用，在编译时不可用的注解。

# 3. 消费器

注解消费器：利用被注解代码并根据注解信息产生不同行为的系统或者应用程序。

例如，在Java自带的内建注解（元注解）中，消费器是执行被注解代码的JVM。例如JUnit，消费器是读取，分析被注解代码的JUnit处理程序，它还可以决定测试单元和方法执行顺序。消费器使用Java中的反射机制来读取和分析被注解的源代码。使用的主要的包有：java.lang, java.lang.reflect。

# 4. 注解语法和元素

声明一个注解需要使用“@”作为前缀，这便向编译器说明，该元素为注解。例如：

	@Annotation
	public void annotatedMethod() {
		...
	}
	
注解属性：

	@Annotation(
		info = "I am an annotation",
		counter = "55"
	)
	public void annotatedMethod() {
		...
	}
	
如果注解只包含一个元素（或者只需要指定一个元素的值，其它则使用默认值），可以像这样声明：

	@Annotation("I am an annotation")
	public void annotatedMethod() {
		...
	}
	
多个注解可以使用在同一代码上，例如类：

	@Annotation(info = "UauO")
	@Annotation2
	class AnnotatedClass { ... }
	
一些java本身提供的开箱即用的注解，我们称之为内建注解。也可以定义你自己的注解，称之为子定义注解。

# 5. 在什么地方使用

注解基本上可以在Java程序的每一个元素上使用：类，域，方法，包，变量，等等。

自Java8，诞生了通过类型注解的理念。在此之前，注解是限于在前面讨论的元素的声明上使用。从此，无论是类型还是声明都可以使用注解，就像：

	@MyAnnotation String str = "danibuiza";
	
# 6. 使用案例

注解可以满足许多要求，最普遍的是：

*	向编译器提供信息：注解可以被编译器用来根据不同的规则产生警告，甚至错误。一个例子是Java8中@FunctionalInterface注解，这个注解使得编译器校验被注解的类，检查它是否是一个正确的函数式接口。
*	文档：注解可以被软件应用程序计算代码的质量例如：FindBugs，PMD或者自动生成报告，例如：用来Jenkins, Jira，Teamcity。
*	代码生成：注解可以使用代码中展现的元数据信息来自动生成代码或者XML文件，一个不错的例子是JAXB。
*	运行时处理：在运行时检查的注解可以用做不同的目的，像单元测试（JUnit），依赖注入（Spring），校验，日志（Log4j），数据访问（Hibernate）等等。

# 7. 内建注解

@Rentention：这个注解注在其他注解上，并用来说明如何存储已被标记的注解。这是一种元注解，用来标记注解并提供注解的信息。可能的值是：

*	SOURCE：表明这个注解会被编译器忽略，并只会保留在源代码中。
*	CLASS：表明这个注解会通过编译驻留在CLASS文件，但会被JVM在运行时忽略,正因为如此,其在运行时不可见。
*	RUNTIME：表示这个注解会被JVM获取，并在运行时通过反射获取。

@Target：这个注解用于限制某个元素可以被注解的类型。例如：

*	ANNOTATION_TYPE 表示该注解可以应用到其他注解上
*	CONSTRUCTOR 表示可以使用到构造器上
*	FIELD 表示可以使用到域或属性上
*	LOCAL_VARIABLE表示可以使用到局部变量上。
*	METHOD可以使用到方法级别的注解上。
*	PACKAGE可以使用到包声明上。
*	PARAMETER可以使用到方法的参数上。
*	TYPE可以使用到一个类的任何元素上。

@Documented：被注解的元素将会作为Javadoc产生的文档中的内容。注解都默认不会成为成为文档中的内容。这个注解可以对其它注解使用。

@Inherited：在默认情况下，注解不会被子类继承。被此注解标记的注解会被所有子类继承。这个注解可以对类使用。

@Deprecated：说明被标记的元素不应该再度使用。这个注解会让编译器产生警告消息。可以使用到方法，类和域上。相应的解释和原因，包括另一个可取代的方法应该同时和这个注解使用。

@SuppressWarnings：说明编译器不会针对指定的一个或多个原因产生警告。例如：如果我们不想因为存在尚未使用的私有方法而得到警告可以这样做：

	@SuppressWarnings("unused")
	private String myNotUsedMethod(){
		...
	}
	
通常,编译器会因为没调用该方而产生警告; 用了注解抑制了这种行为。该注解需要一个或多个参数来指定抑制的警告类型。

@Override：向编译器说明被注解元素是重写的父类的一个元素。在重写父类元素的时候此注解并非强制性的，不过可以在重写错误时帮助编译器产生错误以提醒我们。比如子类方法的参数和父类不匹配，或返回值类型不同。

@SafeVarargs：断言方法或者构造器的代码不会对参数进行不安全的操作。在Java的后续版本中，使用这个注解时将会令编译器产生一个错误在编译期间防止潜在的不安全操作。

更多信息请参考：[http://docs.oracle.com/javase/7/docs/api/java/lang/SafeVarargs.html](http://docs.oracle.com/javase/7/docs/api/java/lang/SafeVarargs.html)

# 8. Java 8与注解

@Repeatable：说明该注解标识的注解可以多次使用到同一个元素的声明上。

	