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

自Java8开始，我们可以在类型上使用注解。由于我们在任何地方都可以使用类型，包括 new操作符，casting，implements，throw等等。注解可以改善对Java代码的分析并且保证更加健壮的类型检查。

	@SuppressWarnings("unused")
	public static void main(String[] args) {
		
		// type def
		@TypeAnnotated
		String cannotBeEmpty = null;
		
		// type 
		List<@TypeAnnotated String> myList = new ArrayList<String>();
		
		// values
		String myString = new @TypeAnnotated String("this is annotated in java 8");
	}
	
	// in method params
	public void methodAnnotated(@TypeAnnotated int parameter) {
		System.out.println("do nothing");
	}
	
@FunctionalInterface：这个注解表示一个函数式接口元素。函数式接口是一种只有一个抽象方法（非默认）的接口。编译器会检查被注解元素，如果不符，就会产生错误。

这个注解可以被使用到类，接口，枚举和注解本身。它的被JVM保留并在runtime可见，这个是它的声明：

	@Documented
	@Rentention(value=RUNTIME)
	@Target(value=TYPE)
	public @interface FunctionalInterface
	
# 9. 自定义注解

有很多其它属性可以用在自定义注解上，但是 目标 （Target）和 保留策略（Retention Policy）是最重要的两个

# 10. 获取注解

*	getAnnotations()：返回该元素的所有注解，包括没有显示定义该元素上的注解。
*	isAnnotationPresent(annotation)：检查传入的注解是否存在于当前元素
*	getAnnotation(class)：按照传入的参数获取指定类型的注解。返回null说明当前元素不带有此注解。

# 11. 注解中的继承

如果一个注解在Java中被标识成继承，使用了保留注解@Inherited，说明它注解的这个类将自动地把这个注解传递到所有子类中而不用在子类中声明。通常，一个类继承了父类，并不继承父类的注解。这完全和使用注解的目的一致的：提供关于被注解的代码的信息而不修改它们的行为。

	@Inherited
	@Retention(RetentionPolicy.RUNTIME)
	@Target(ElementType.TYPE)
	public @interface InheritedAnnotation {
	}
	
继承注解和接口在一起使用时，接口中的注解在实现类中：仅仅被忽略。实现类并不继承接口的注解；接口继承仅仅适用于类继承。

@Inheriated注解仅在存在继承关系的类上产生效果，在接口和实现类上并不工作。这条同样也适用在方法，变量，包等等。只有类才和这个注解连用。

一条关于@Inheriated注解的很好的解释在Javadoc中：[http://docs.oracle.com/javase/7/docs/api/java/lang/annotation/Inherited.html](http://docs.oracle.com/javase/7/docs/api/java/lang/annotation/Inherited.html)

注解不能继承注解。

# 12. 使用注解的知名类库

## 12.1 JUnit

*	@Test：这个注解向JUnit说明这个被注解的方法一定是一个可执行的测试方法。这个注解只能标识在方法上，并且被JVM保留至运行时。
*	@Before：这个注解用来向JUnit说明被标记的方法应该在所有测试方法之前被执行。这对于在测试之前设置测试环境和初始化非常有用。同样只适用于方法上。
*	@After：这个注解用来向JUnit说明被注解的方法应该在所有单元测试之后执行。这个注解通常用来销毁资源，关闭，释放资源或者清理，重置等工作。
*	@Ignore：这个方法用来向JUnit说明被注解的方法应该不被当作测试单元执行。即使它被注解成为一个测试方法，也只能被忽略。
*	@FixMethodOrder：指定执行的顺序，正常情况下Junit处理程序负责它按照完全随机的无法预知的顺序执行。当所有的测试方法都相互独立的时候，不推荐使用这个注解。但是，当测试的场景需要测试方法按照一定规则的时候，这个注解就派上用场了。

还有一些测试套件和类库使用了注解，例如Mockito和JMock，使用注解来创建测试对象和预期值。

## 12.2 Hibernate ORM

@Entity、@Table、@Id、@GeneratedValue、@Column

## 12.3 Spring MVC

*	@Component：说明被标记的元素，在本例中是一个类，是一个自动检测的目标。这意味着被注解的类，将会被Spring容器实例化并管理。
*	@Autowired：Spring容器将会尝试通过类型（这是一种元素匹配机制）使用这个set方法来自动装配。此注解也可以使用在构造器和属性上，Spring也会根据注解的地方不同采取不同的操作。

## 12.4 Findbugs

这是一个用来测量代码质量，并提供一系列可能提高它的工具。它会根据预定义（或者自定义）的违反规则来检查代码。Findbugs提供一系列注解来允许开发者来改变默认行为。

它主要使用反射读取代码（和包含的注解）并决定基于它们，应该采取什么行为。

## 12.5 JAXB

JAXB是一个用来相互转换和映射XML文件与Java对象的类库。实际上，这个类库与标准JRE一起提供，不需要任何额外的下载和配置。可以直接通过引入 java.xml.bind.annotation 包下的类直接使用。

JAXB使用注解来告知处理程序（或者是JVM）XML文件与代码的相互转化。

	@XmlType( propOrder = { "brand", "model", "year", "km" } )
	@XmlRootElement( name = "Car" )
	class Car
	...
	
使用注解@XmlType，@XmlRoootElement。它们用来告知 JAXB 处理程序 Car 这个类在转换后，将会转换成为XML中的一个节点。这个@XmlType说明了属性在XML中的顺序。JAXB将会基于这些注解执行合适的操作。

# 13. 总结

# 14. 下载

# 15. 资料

[官方Java注解地址](http://docs.oracle.com/javase/tutorial/java/annotations/)

[维基百科中关于Java注解的解释](http://en.wikipedia.org/wiki/Java_annotation)

[Java规范请求250](http://en.wikipedia.org/wiki/JSR_250)

[Oracle 注解白皮书](http://www.oracle.com/technetwork/articles/hunter-meta-096020.html)

[注解API](http://docs.oracle.com/javase/7/docs/api/java/lang/annotation/package-summary.html)