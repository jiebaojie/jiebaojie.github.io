---
layout: post
notes: true
subtitle: "【技术博客】API Design with Java8"
comments: false
author: "Per-Ake Minborg（码农网-小峰译）"
date: 2017-07-03 00:00:00

---


原文：[http://www.codeceo.com/article/java-8-api-design.html/](http://www.codeceo.com/article/java-8-api-design.html/)
译文：[https://dzone.com/articles/the-java-8-api-design-principles/](https://dzone.com/articles/the-java-8-api-design-principles/)

*   目录
{:toc }

# 不要用返回Null来表示一个空值

所以Optional中我们真正可以来的应该是除了 isPresent() 和 get() 的其他方法。可以证明，不一致的null处理（导致无处不在的NullPointerException）是历史上Java应用程序错误最大的唯一来源。

你可以这样写

	public Optional<String> getComment() {
		return Optional.ofNullable(comment);
	}
	
而不要这样写

	public String getComment() {
		return comment;		// comment is nullable
	}
	
# 不要将数组作为API的传入参数和返回值

在一般情况下，如果API要返回一组元素，考虑公开Stream。这清楚地说明了结果是只读的（与具有set()方法的List相反）。

它还允许客户端代码容易地收集另一个数据结构中的元素或在运行中对它们进行操作。此外，API可以在元素变得可用时（例如，从文件，套接字或从数据库中拉入），延迟生成元素。同样，Java 8改进的转义分析将确保在Java堆上创建实际最少的对象。

也不要使用数组作为方法的输入参数，因为——除非创建数组的保护性副本——使得有可能另一个线程在方法执行期间修改数组的内容。

你可以这样写

	public Stream<String> comments() {
		return Stream.of(comments);
	}
	
而不要这样写

	public String[] comments() {
		return comments;	// Exposes the backing array!
	}
	
# 考虑添加静态接口方法以提供用于对象创建的单个入口点

避免允许客户端代码直接选择接口的实现类。

考虑添加静态接口方法，以允许客户端代码来创建（可能为专用的）实现接口的对象。例如，如果我们有一个接口Point有两个方法int x() 和int y() ，那么我们可以显示一个静态方法Point.of( int x，int y) ，产出接口的（隐藏）实现。



你可以这样写

	Point point = Point.of(1, 2);
	
而不要这样写

	Point point = new PointImpl(1, 2);
	
所以，如果x和y都为零，那么我们可以返回一个特殊的实现类PointOrigoImpl（没有x或y字段），否则我们返回另一个保存给定x和y值的类PointImpl。

# 青睐功能性接口和Lambdas的组合优于继承

出于好的原因，对于任何给定的Java类，只能有一个超类。此外，在API中展示抽象或基类应该由客户端代码继承，这是一个非常大和有问题的API 功能。避免API继承，而考虑提供静态接口方法，采用一个或多个lambda参数，并将那些给定的lambdas应用到默认的内部API实现类。

这也创造了一个更清晰的关注点分离。例如，并非继承公共API类AbstractReader和覆盖抽象的空的handleError（IOException ioe），我们最好是在Reader接口中公开静态方法或构造器，接口使用Consumer <IOException>并将其应用于内部的通用ReaderImpl。

你可以这样写

	Reader reader = Reader.builder()
		.withErrorHandler(IOException::printStackTrace)
		.build();
		
而不要这样写

	Reader reader = new AbstractReader() {
		@Override
		public void handleError(IOException ioe) {
			ioe. printStackTrace();
		}
	};
	
# 确保你将@FunctionalInterface注解添加到功能性接口

使用@FunctionalInterface注解标记的接口，表示API用户可以使用lambda实现接口，并且还可以通过防止抽象方法随后被意外添加到API中来确保接口对于lambdas保持长期使用。

# 避免使用功能性接口作为参数的重载方法

如果有两个或更多的具有相同名称的函数将功能性接口作为参数，那么这可能会在客户端侧导致lambda模糊。例如，如果有两个Point方法add(Function<Point, String> renderer) 和add(Predicate<Point> logCondition)，并且我们尝试从客户端代码调用point.add(p -> p + “ lambda”) ，那么编译器会无法确定使用哪个方法，并产生错误。相反，请根据具体用途考虑命名方法。

你可以这样写

	public interface Point {
		addRenderer(Function<Point, String> renderer);
		addLogCondition(Predicate<Point> logCondition);
	}
	
而不要这样写

	public interface Point {
		add(Function<Point, String> renderer);
		add(Predicate<Point> logCondition);
	}
	
# 避免在接口中过度使用默认方法

默认方法可以很容易地添加到接口，有时这是有意义的。例如，想要一个对于任何实现类都期望是相同的并且在功能上要又短又“基本”的方法，那么一个可行的候选项就是默认实现。此外，当扩展API时，出于向后兼容性的原因，提供默认接口方法有时是有意义的。

众所周知，功能性接口只包含一个抽象方法，因此当必须添加其他方法时，默认方法提供了一个安全舱口。然而，通过用不必要的实现问题来污染API接口以避免API接口演变为实现类。如果有疑问，请考虑将方法逻辑移动到单独的实用程序类和/或将其放置在实现类中。

# 确保在执行之前进行API方法的参数不变量检查

确保在实现类中使用参数之前检查参数的空值和任何有效的范围约束或前提条件。不要因性能原因而跳过参数检查的诱惑。

JVM能够优化掉冗余检查并产生高效的代码。好好利用Objects.requireNonNull()方法。参数检查也是实施API约定的一个重要方法。如果不想API接受null但是却做了，用户会感到困惑。

# 不要简单地调用Optional.get()

Java 8的API设计师犯了一个错误，在他们选择名称Optional.get()的时候，其实应该被命名为Optional.getOrThrow()或类似的东西。调用get()而没有检查一个值是否与Optional.isPresent()方法同在是一个非常常见的错误，这个错误完全否定了Optional原本承诺的null消除功能。考虑在API的实现类中使用任一Optional的其他方法，如map()，flatMap()或ifPresent()，或者确保在调用get()之前调用isPresent()。

你可以这样写

	Optional<String> comment = // some Optional value 
	String guiText = comment
		.map(c -> "Comment: " + c)
		.orElse("");
		
而不要这样写
	
	Optional<String> comment = // some Optional value 
	String guiText = "Comment: " + comment.get();
	
# 考虑在不同的API实现类中分行调用接口

最后，所有API都将包含错误。当接收来自于API用户的堆栈跟踪时，如果将不同的接口分割为不同的行，相比于在单行上表达更为简洁，而且确定错误的实际原因通常更容易。此外，代码可读性将提高。