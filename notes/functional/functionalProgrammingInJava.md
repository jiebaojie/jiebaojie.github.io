---
layout: post
notes: true
subtitle: "Functional Programming in Java"
comments: false
author: "[法] Pierre-Yves Saumont 著 （高清华 译）"
date: 2018-01-28 00:00:00

---


![](/img/notes/functional/functionalProgrammingInJava/functional_programming_in_java.jpg)

*   目录
{:toc }

# 1. 什么是函数式编程

本章要点：

*	函数式编程的优势
*	副作用的问题
*	引用透明如何让线程更安全
*	使用代换模型编程的原因
*	充分利用抽象

一般来说，函数式编程是用函数来编程的一种编程范式。

John Hughes写到：

*函数式编程中没有赋值语句，因此变量一旦有了值，就不会再改变了。更通俗地说，函数式编程完全没有副作用。除了计算结果，调用函数没有别的作用。这样便消除了bug的一个主要来源，也使得执行顺序变得无关紧要——因为没有能够改变表达式值的副作用，可以在任何时候对它求值。这样便把程序员从处理控制流程的负担中解脱出来。由于能够在任何时候对表达式求值，所以可以用变量的值来自由替换表达式，反之亦然——即程序式“引用透明”的。这样的自由度让函数式的程序比它们的一般对手在数学上更易驾驭。

## 1.1 函数式编程是什么

函数式编程有时被认为是一系列可以补充或替代其他编程范式的技术，例如：

*	函数是一等公民
*	匿名函数
*	闭包
*	柯里化
*	惰性求值
*	参数多态
*	代数数据类型

命令式编程和函数式编程的一个最大的不同是，函数式编程没有副作用。这意味着其中：

*	没有变量改变
*	没有打印到控制台或其他设备
*	没有写入文件、数据库、网络或其他什么
*	没有抛出异常

函数就是如下黑盒：

*	接收一个参数（一个单独的参数）
*	内部做一些神秘的事情，例如改变变量的值，还有许多命令式风格的东西，但是在外界来看并没有什么作用
*	返回一个（单独的）值

函数式编程其实是编写**非故意的副作用**的程序，副作用是程序预期结果的一部分。非故意的副作用也应该越少越好。

## 1.2 编写没有副作用的程序

*	异常的副作用
*	日志的副作用

## 1.3 引用透明如何让程序更安全

不被外界所影响的代码就是引用透明的，引用透明的代码的性质：

*	它是独立的，并不依赖于任何外部的设备来工作
*	它是确定的，意味着相同的参数总是返回相同的结果
*	它绝对不会抛出任何种类的Exception。它可能抛出错误，例如OOME（out-of-memory error，即内存耗尽）或是SOE（stack-overflow error，即堆栈溢出），但是这些错误表示代码有bug，并不是做作为程序员的你或是你API的用户应该处理的。
*	任何时候它都不会导致其他代码意外失败。
*	它不会由于外部设备（数据库、文件系统或网络）不可用、太慢或坏掉而崩溃。

## 1.4 函数式编程的优势

*	函数式程序更加易于推断，因为它们是确定性（deterministic）的。对于一个特定的输入总会给出相同的输出。
*	函数式程序更加易于测试。
*	函数式程序更加模块化，因为它们是由只有输入和输出的函数构建的。
*	函数式编程让复合和重新复合更加简单。

函数式程序天生就是线程安全的。

## 1.5 用代换模型来推断程序

请记住一个函数什么事情都不做。它只是有一个值，依赖于参数而已。因此，永远都可以用其值来替换一个函数调用或任何引用透明的表达式。

## 1.6 将函数式原则应用于一个简单的例子

## 1.7 抽象到极致

## 1.8 总结

*	函数式编程就是用函数来编程，返回结果，没有任何副作用
*	函数式程序容易推断，也容易测试
*	函数式编程提供了高层次的抽象和重用
*	函数式程序比命令式程序更加健壮
*	函数式程序由于避免了共享可变状态，因而在多线程环境中更加安全

# 2. 在Java中使用函数

本章要点：

*	理解现实世界中的函数
*	在Java中表示函数
*	使用lambda
*	使用高阶函数
*	使用柯里化函数
*	用函数式接口编程

## 2.1 什么是函数

### 2.1.1 现实世界里的函数

函数定义域（function domain）的源集（source set）和被称为函数陪域（function codomain）的目标集（target set）之间的关系。

#### 如何让两个集之间的关系成为函数

为了成为函数，关系需要满足一个条件：定义域内的所有元素都必须在陪域内有且仅有一个对应元素。这个条件有一些有趣的含义：

*	定义域里不存在在陪域里没有对应值的元素
*	定义域里的一个元素在陪域里不会有两个对应的元素
*	陪域里的元素可能在源集里没有相对应的元素
*	陪域里的元素可能在源集里有多少个相对应的元素
*	在定义域里存在对应元素的陪域元素集被称为函数的像（image of the function）

#### 逆函数

函数未必会有逆函数（inverse function）。如果你用A->B来表示函数，那么逆函数（如果存在的话）就是B->A。

#### 偏函数

没有在定义域中定义所有元素但是满足其他需求（定义域里不存在任何在陪域里有多个元素与之相对应的元素）的关系一般称为偏函数（partial function）。如果你用A-

例如，f(x) = 1/x 是一个从N到Q（有理数）的偏函数，因为它对0没有定义。它是一个从N*到Q的全函数，也是一个从N到（Q与error）的全函数。通过在陪域里增加一个元素（错误条件），可以将偏函数转化为全函数。但是这样做的话，这个函数需要有一种返回错误的方法。把偏函数转换成全函数是函数式编程里的一个重要组成部分。

#### 复合函数

函数就像积木，可以复合为其他函数。函数f和g的复合函数标记为f ○ g，读作f round g。

f ○ g (x)和f(g(x))式等价的。

应用函数的顺序与函数的顺序正好相反。如果你写f ○ g，首先应用g然后才是f。标准的Java 8函数定义了compose()和andThen()方法来表示这两种例子（f.andThen(g)与g.compose(f)或g ○ f等价）。

#### 多参函数

一个函数不允许有多个参数，但是两个集的乘积本身就是一个集，所以这样的函数可能确实有多个参数。

#### 函数柯里化

函数f(x)(y)式f(x,y)的柯里化形式。对一个元组函数（多参函数）应用这种转换就称为柯里化（currying）。

#### 偏应用函数

函数f(rate, price)：f(rate)和g(price)这样的函数被称为偏应用函数。

#### 没有作用的函数

纯函数只会返回一个值，不会做任何其他事情。

## 2.2 Java中的函数

### 2.2.1 函数式方法

一个方法可以式函数式的，只要它满足纯函数的要求：

*	它不能修改函数外的任何东西。外部观测不到内部的任何变化。
*	它不能修改自己的参数。
*	它不能抛出错误或异常。
*	它必须返回一个值。
*	只要调用它的参数相同，结果也必须相同。

#### 对象标记与函数标记

	Payment newPayment = p0.combine(p1).combine(p2).combine(p3);

### 2.2.2 Java的函数式接口与匿名类

### 2.2.3 复合函数

	Function compose(final Function f1, final Function f2) {
		return new Function() {
			@Override
			public int apply(int arg) {
				return f1.apply(f2.apply(arg));
			}
		};
	}
	
### 2.2.4 多态函数

### 2.2.5 通过lambda简化代码

## 2.3 高级函数特性

### 2.3.1 多参函数怎么样

### 2.3.2 应用柯里化函数

### 2.3.3 高阶函数

接收函数为参数并返回函数，被称为高阶函数（higher-order function，即HOF）

### 2.3.4 多态高阶函数

### 2.3.5 使用匿名函数

### 2.3.6 局部函数

### 2.3.7 闭包

lambda还有附加需求：一个lambda访问的局部变量必须式final的。这并不是lambda的新要求，而是Java 8以前的版本对匿名类的要求，而lambda也需要遵守相同的约定，虽然它并没有那么严格。自Java 8起，从匿名类或是lambda访问的元素都是隐式final的；它们无须被声明为final来表明它们不会被改变的。

### 2.3.8 部分函数应用和自动柯里化

柯里化包括了把接收元组的函数替换为可以部分应用各个参数的函数。这是柯里化函数和元组函数最主要的区别。使用元组函数，所有的参数都在调用函数之前就计算出来了。使用柯里化版本，所有的参数都必须在完全应用函数之前确定，但是每个单独的参数都可以在函数部分应用它之前才计算。你不必将函数完全柯里化。一个接收三个参数的函数可以被柯里化为一个生成单参函数的二元组函数。

### 2.3.9 交换部分应用函数的参数

	public static <T, U, V> Function<U, Function<T, V>> reverseArgs(Function<T, Function<U, V>> f) {
		return u -> t -> f.apply(t).apply(u);
	}
	
### 2.3.10 递归函数

	public final Function<Integer, Integer> factorial = n -> n <= 1 ? n : n * this.factorial.apply(n - 1);
	
	public static final Function<Integer, Integer> factorial = n -> n <= 1 ? n : n * FunctionExamples.factorial.apply(n - 1);
	
### 2.3.11 恒等函数

## 2.4 Java 8 的函数式接口

## 2.5 调试lambda

## 2.6 总结

*	函数是源集和目标集之间的关系。它在源集的元素（定义域）和目标集的元素（陪域）之间建立联系。
*	纯函数除了返回一个值以外，没有任何可观测的作用。
*	函数只有一个参数，参数可能是一个元组，元组内包含着多个元素。
*	元组函数可以被柯里化，以便能够一次应用元组里的一个元素。
*	当柯里化函数仅被应用于一部分参数时，我们称其被部分应用了。
*	在Java里，函数可以表现为方法、lambda、方法应用或是匿名类。
*	方法引用是函数的最佳表现形式。
*	函数可以复合为新函数。
*	函数可以递归调用它自己，但是递归的深度受限于栈的大小。
*	lambda和方法引用可以用于需要函数式接口的地方。

# 3. 让Java更加函数式

本章要点：

*	让标准控制结构具有函数式风格
*	抽象控制结构
*	抽象迭代
*	使用正确的类型

## 3.1 使标准控制结构具有函数式风格

## 3.2 抽象控制结构

### 3.2.1 清理代码

	public interface Effect<T> {
		
		void apply(T t);
	}
	
	public interface Result<T> {
	
		void bind(Effect<T> success, Effect<String> failure);
		
		public static<T> Result<T> failure(String message) {
			return new Failure<>(message);
		}
		
		public static <T> Result<T> success(T value) {
			return new Success<>(value);
		}
		
		public class Success<T> implements Result<T> {
			
			private final T value;
			
			private Success(T t) {
				value = t;
			}
			
			@Override
			public void bind(Effect<T> success, Effect<String> failure) {
				success.apply(value);
			}
		}
		
		public class Failure<T> implements Result<T> {
			
			private final String errorMessage;
			
			private Failure(String s) {
				this.errorMessage = s;
			}
			
			@Override
			public void bind(Effect<T> success, Effect<String> failure) {
				failure.apply(errorMessage);
			}
		}
	}
	
### 3.2.2 if ... else的另一种方式

	public class Case<T> extends Tuple<Supplier<Boolean>, Supplier<Result<T>>> {
		
		private Case(Supplier<Boolean> booleanSupplier, Supplier<Result<T>> resultSupplier) {
			super(booleanSupplier, resultSupplier);
		}
	
		private static class DefaultCase<T> extends Case<T> {
			private DefaultCase(Supplier<Boolean> booleanSupplier, Supplier<Result<T>> resultSupplier) {
				super(booleanSupplier, resultSupplier);
			}
		}
	
		public static <T> Case<T> mcase(Supplier<Boolean> condition, Supplier<Result<T>> value) {
			return new Case<>(condition, value);
		}
	
		public static <T> DefaultCase<T> mcase(Supplier<Result<T>> value) {
			return enw DefaultCase<>(() -> true, value);
		}
		
		@SafeVarags
		public static <T> Result<T> match(DefaultCase<T> defaultCase, Case<T> ... matchers) {
			for (Case<T> aCase : matchers) {
				if (aCase._1.get()) {
					return aCase._2.get();
				}
				return defaultCase._2.get();
			}
		}
	}
	
## 3.3 抽象迭代

你可能想在循环里做以下几件事情：

*	将每个元素转换为其他元素。
*	将元素聚合成单个结果。
*	根据元素自身的条件删除一些元素。
*	根据外部条件删除一些元素。
*	根据某些条件分组元素。

### 3.3.1 使用映射抽象列表操作

### 3.3.2 创建列表

	public static <T> List<T> list() {
        return Collections.emptyList();
    }

    public static <T> List<T> list(T t) {
        return Collections.singletonList(t);
    }

    public static <T> List<T> list(List<T> ts) {
        return Collections.unmodifiableList(new ArrayList<>(ts));
    }

    public static <T> List<T> list(T... t) {
        return Collections.unmodifiableList(Arrays.asList(Arrays.copyOf(t, t.length)));
    }
	
### 3.3.3 使用head和tail操作

对列表的函数式操作经常会访问列表的head（或第一个元素）以及tail（删除第一个元素后的列表）。

	public static <T> Try<T> head(List<T> list) {
        return Match.of(list).cases(
                l -> Try.success(l.get(0)),
                Case.of(l -> l.size() == 0, Try.failure(new IllegalStateException("head of empty list")))
        );
    }
	
	private static <T> List<T> copy(List<T> ts) {
        return new ArrayList<>(ts);
    }
	
	public static <T> Try<List<T>> tail(List<T> list) {

        DefaultCase<List<T>, Try<List<T>>> defaultCase = l -> {
            List<T> workList = copy(l);
            workList.remove(0);
            return Try.success(Collections.unmodifiableList(workList));
        };

        return Match.of(list).cases(
                defaultCase,
                Case.of(l -> l.size() == 0, Try.failure(new IllegalStateException("tail of empty list")))
        );
    }
	
### 3.3.4 函数式地添加列表元素

	public static <T> List<T> append(List<T> list, T t) {
        List<T> ts = copy(list);
        ts.add(t);
        return Collections.unmodifiableList(ts);
    }
	
### 3.3.5 化简和折叠列表

列表折叠（fold）通过使用一个特定操作来将列表转换为单值。

可以从左到右或从右到左两个方向上折叠列表，视所用的操作而定：

*	如果操作可交换，则两种折叠方式是等价的
*	如果操作不可交换，则两种折叠方式会给出不同的结果

折叠操作需要一个起始值，即操作的中性元素或单位元（identity element），该元素作为*累加器*（accumulator）的起始值。当计算完成时，累加器内即包含结果。另一方面，只要列表不为空，也可以在没有起始元素的情况下执行化简，因为第一个（或最后一个）元素将作为起始元素。

左折叠：

	public static <T, U> U foldLeft(List<T> ts, U identity, Function<U, Function<T, U>> f) {

        U result = identity;
        for (T t : ts) {
            result = f.apply(result).apply(t);
        }
        return result;
    }
	
右折叠：

	public static <T, U> U foldRight(List<T> ts, U identity, Function<T, Function<U, U>> f) {

        U result = identity;
        for (int i = ts.size() - 1; i >= 0; i--) {
            result = f.apply(ts.get(i)).apply(result);
        }
        return result;
    }
	
反转列表：

	public static <T> List<T> reverse(List<T> list) {

        List<T> result = new ArrayList<>(list.size());
        for (int i = list.size() - 1; i >= 0; i--) {
            result.add(list.get(i));
        }
        return Collections.unmodifiableList(result);
    }
	
### 3.3.6 复合映射和映射复合

映射复合而不是复合映射

### 3.3.7 对列表应用作用

