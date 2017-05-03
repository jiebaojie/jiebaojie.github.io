---
layout: post
notes: true
subtitle: "Scala for the Impatient"
comments: false
author: "[美] Cay S. Horstmann 著 （高宇翔 译）"
date: 2017-04-24 00:00:00

---


![](/img/notes/functional/scalaForTheImpatient/scala.jpg)

*   目录
{:toc }

# 第1章 基础

## 1.1 Scala解释器

启动Scala解释器的步骤如下：

*	安装Scala
*	确保scala/bin目录位于系统PATH中
*	在你的操作系统中打开命令行窗口
*	键入scala并按Enter键

示例：

	scala> 8 * 5 + 2
	res0: Int = 42
	scala> 0.5 * res0
	res1: Double = 21.0
	scala> "Hello," + res0
	res2: java.lang.String = Hello, 42

从技术上讲，scala程序并不是一个解释器。实际发生的是，你输入的内容被快速地编译成字节码，然后这段字节码交由Java虚拟机执行。正因如此，大多数Scala程序员更倾向于将它称做"REPL"。

## 1.2 声明值和变量

	scala> val answer = 8 * 5 + 2
	answer: Int = 42
	scala> 0.5 * answer
	res3: Double = 21.0

以val定义的值实际上是一个常量——你无法改变它的内容。

如果要声明其值可变的变量，可以用var。

在Scala中，我们鼓励你使用val——除非你真的需要改变它的内容。

在必要的时候，可以指定变量类型：

	val greeting: String = null

在Scala中，仅当同一行代码中存在多条语句时才需要用分号隔开。

你可以将多个值或变量放在一起声明：

	val xmax, ymax = 100
	val greeting, message: String = null

## 1.3 常用类型

Scala的类型都是类，Scala并不刻意区分基本类型和引用类型。你可以对数字执行方法，例如：

	1.toString()    // 产出字符串"1"
	1.to(10)        // 产出Range(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)

在Scala中，我们不需要包装类型。在基本类型和包装类型之间的转换是Scala编译器的工作。举例来说，如果你创建一个Int的数组，最终在虚拟机中得到的是一个int[]数组。

Scala用底层的java.lang.String类来表示字符串。不过，它通过StringOps类给字符串追加了上百种操作。

举例来说，intersect方法输出两个字符串共通的一组字符：

	"Hello".intersect("World")

在这个表达式中，java.lang.String对象"Hello"被隐式地转换成了一个StringOps对象，接着StringOps类的intersect方法被应用。

同样地，Scala还提供了RichInt、RichDouble、RichChar等。它们分别提供了它们可怜的堂兄弟们——Int、Double、Char等所不具备的便捷方法。我们前面用到的to方法事实上就是RichInt类中的方法。在表达式：

	1.to(10)

中，Int值1首先被转换成RichInt，然后再应用to方法。

最后，还有BigInt和BigDecimal类，用于任意大小（但有穷）的数字。这些类背后分别对应的是java.math.BigInteger和java.math.BigDecimal，不过，它们用起来更加方便，你可以用常规的数学操作符来操作它们。

## 1.4 算术和操作符重载

Scala的操作符实际上是方法，如：

	a + b

是如下方法调用的简写：

	a.+(b)

通常来说，你可以用：

	a 方法 b

作为以下代码的简写：

	a.方法(b)

Scala并没有提供++和--操作符，我们需要使用+=1或者-=1：

	counter += 1

对于Bigint和BigDecimal对象，你可以以常规的方式使用那些数学操作符：

	val x: BigInt = 1234567890
	x * x * x 

## 1.5 调用函数和方法

相比Java，在Scala中使用数学函数（比如min或pow）更为简单——你不需要从某个类调用它的静态方法。

	sqrt(2)
	pow(2, 4)
	min(3, Pi)

这些数学函数是在scala.math包中定义的。你可以用如下语句进行引入：

	import scala.math._    // 在Scala中，_字符是“通配符”，类似Java中的*

使用以scala.开头的包时，我们可以省去scala的前缀。例如，import math._等同于import scala.math._，而math.sqrt(2)等同于scala.math.sqrt(2)。

Scala没有静态方法，不过它有个类似的特性，叫做单例对象（singleton object）。通常，一个类对应有一个伴生对象（companion object）。举例来说，BigInt类的BigInt伴生对象有一个生成指定位数的随机素数的方法probablePrime：

	BigInt.probablePrime(100, scala.util.Random)

这里的Random是一个单例的随机数生成器对象。这是用单例对象比用类更好的为数不多的场景之一。在Java中，为每个随机数都构造出一个新的java.util.Random对象是一个常见的错误。

不带参数的scala方法通常不使用圆括号：

	"Hello".distinct

## 1.6 apply方法

	"Hello"(4)

是如下语句的缩写：

	"Hello".apply(4)

又如：
	
	BigInt("1234567890")

是如下语句的缩写：
	
	BigInt.apply("1234567890")

这个语句产出一个新的BigInt对象，不需要使用new。

像这样使用伴生对象的Apply方法是Scala中构建对象的常用方法。例如，Array(1, 4, 9, 16)返回一个数组，用的就是Array伴生对象的apply方法。

## 1.7 Scaladoc

以下这些小窍门：

*	如果你想使用数值类型，记得看看RichInt、RichDouble等。同理，如果想使用字符串，记得看看StringOps。
*	那些数学函数位于scala.math包中，而不是位于某个类中。
*	有时你会看到名称比较奇怪的函数。比如BigInt有一个方法叫做unary_-，这就是你定义前置的负操作符-x的方式。
*	标记为implicit的方法对应的是自动（隐士）转换。
*	方法可以以函数作为参数。
*	你时不时地会遇到类似Range或Seq[Char]这样的类。它们的含义和你的直觉告诉你的一样：一个是数字区间，一个是字符序列。

# 第2章 控制结构和函数

## 2.1 条件表达式

在Scala中if/else表达式有值，这个值就是跟在if或else之后的表达式的值。例如：

	if (x > 0) 1 else -1

你可以将if/else表达式的值赋值给变量：

	val s = if (x > 0) 1 else -1

在Scala中，每个表达式都有一个类型。以下是混合类型表达式：

	if (x > 0) "positive" else -1

上述表达式的公共超类型叫做Any。

Scala引入一个Unit类，写做()，如：

	if (x > 0) 1 else ()

你可以把()当做是表示“无有用值”的占位符，将Unit当做Java或C++中的void。

Scala没有switch语句，不过它有一个强大得多的模式匹配机制。

## 2.2 语句终止

## 2.3 块表达式和赋值

在Scala中，{}块包含一系列表达式，其结果也是一个表达式。块中最后一个表达式的值就是块的值。

在Scala中，赋值动作本身是没有值的——或者，更严格地说，它们的值是Unit类型的。

一个以赋值语句结束的块，比如：

	{r = r * n; n -= 1}

的值是Unit类型的。

由于赋值语句的值是Unit类型的，别把它们串接在一起。

	x = y = 1    // 别这样做

y = 1的值是()，你几乎不太可能想把一个Unit类型的值赋值给x。

## 2.4 输入和输出

	print("Answer: ")
	println(42)

与下面的代码输出的内容相同：

	println("Answer: " + 42)

另外，还有一个带有C风格格式化字符串的printf函数：

	printf("Hello, %s! You are %d years old. \n", "Fred", 42)

readLine函数从控制台读取一行输入。readLine带一个参数作为提示字符串：

	val name = readLine("Your name: ")
	print ("Your age: ")
	val age = readInt()
	printf("Hello, %s! Next year, your will be %d.\n", name, age + 1)

## 2.5 循环

while循环：

	while (n > 0) {
		r = r * n
		n -= 1
	}

Scala没有与for循环直接对应的结构：

	for (i <- 1 to n)
		r = r * i

下标从0开始：
	
	for (i <- 0 until s.length) 
		sum += s(i)

不需要使用下标：

	var sum = 0
	for (ch <- "Hello") sum += ch

Scala并没有提供break或continue语句来退出循环。那么如果需要break时怎么做？

*	使用Boolean型的控制变量。
*	使用嵌套函数——你可以从函数中return。
*	使用Breaks对象中break方法。

如：

	import scala.util.control.Breaks._
	breakable {
		for (...) {
			if (...) break;    // 退出breakable块
			...
		}
	}

在这里，控制权的转移是通过抛出和捕获异常完成的，因此，如果时间很重要的话，你应该尽量避免使用这套机制。