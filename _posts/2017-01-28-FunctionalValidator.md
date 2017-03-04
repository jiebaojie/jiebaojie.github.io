---
layout: post
title: Java的业务逻辑函数式验证框架functional-validator
comments: true
author: "Bao Jie"
date: 2017-01-28 12:00:00
header-img: 
tags:
    - Java
    - Validator
    - 函数式
---

# 1 背景

functional-validator是具有函数式风格的业务逻辑验证框架，它的创作灵感来自于[fluent-validator](https://github.com/neoremind/fluent-validator)，大家在阅读本文前可以先简单浏览下fluent-validator的[中文手册](http://neoremind.com/2016/02/java%E7%9A%84%E4%B8%9A%E5%8A%A1%E9%80%BB%E8%BE%91%E9%AA%8C%E8%AF%81%E6%A1%86%E6%9E%B6fluent-validator/)。

首先看一个fluent-validator的例子：

	try {
		Result ret = FluentValidator.checkAll()
			.on(car.getLicensePlate(), new CarLicensePlateValidator())
			.on(car.getManufacture(), new CarManufactureValidator())
			.on(car.getSeatCount(), new CarSeatCountValidator())
			.doValidate()
			.result(toSimple());
		if (ret.isSuccess()） {
			System.out.println("success");
		} else {
			System.out.println(ret);
		}
	} catch (RuntimeValidateException e) {
		System.out.println(e.getMessage());
	}
	
由上述代码可以看出，fluent-validator写出的代码非常优雅，且具有很高的可读性和可扩展性，但我在一段时间的项目实践中发现使用该框架有以下几个痛点：

*	需要为每个校验都实现一个校验器类，虽然校验器可以复用，但每一个新的校验器都需要重新写一个Validator类，略显麻烦
*	对校验结果的处理支持的不够友好，需要if-else以及catch异常（或者在校验器中处理异常）进行处理
*	校验对象的NPE问题，如示例中，如果car对象为null，那么直接就NPE了

functional-validator主要借鉴fluent-validator思想，针对以上3个痛点问题，目标提供更优雅、更易用的函数式业务逻辑验证框架。

# 2 上手

github地址：[https://github.com/javadeep/common-functional](https://github.com/javadeep/common-functional)

	String ret = FunctionalValidator.checkFrom(car)
		.failFast()
		.on(Objects::nonNull, "car is null")
		.on(car -> car.getSeatCount() >= 2, "Seat count is not valid")
		.doValidate()
		.fold(car -> "success", result -> result.toString(), e -> e.getMessage());
	System.out.println(ret);
	
代码看着非常优雅简洁吧，和上面的fluent-validator风格的代码相比，你无须创建额外的校验器类，而由对应的函数（Lambda表达式）替换，另外对于校验结果的处理，由fold方法配合3个函数即可优雅地处理可能出现的3种分支情况（校验成功、校验失败、出现异常）。同时，假如car对象为null，则由于car -> car.getSeatCount() >= 2函数 的惰性求值特性，在doValidate()之前不会调用到car.getSeatCount()，也就不会再出现可恶的NPE了（第一个null校验后即failFast）。

再稍微说明下，首先我们通过FunctionalValidator.checkFrom获取了一个FunctionalValidator实例，并且传入校验的对象为car；紧接着调用了failFast()表示有错了立即返回，它的反义词是failOver；然后，一连串的on()操作表示在car对象上依次进行校验，当然，此时真正的校验还并没有开始，直到doValidate()验证才真正执行了，最后我们用fold()方法归约出验证过程中各个分支采取的处理逻辑产生的结果。

最后需要注意的是，使用functional-validator框架必须使用jdk8及以上的版本。

# 3 深入实践

## 3.1 探秘校验器

可以在FunctionalValidator类中找到校验器的定义：

	private final List<Function<? super T, Stream<ValidationError>>> validators = new LinkedList<>();
	
每一个校验器本质上是一个对象到错误集合的一个映射，错误集合为空，则表示校验成功，否则表示校验过程中有错误，校验失败。

FunctionalValidator实例中提供了多种on()校验方式：

*	on(Function<? super T, Stream<ValidationError>> v)：直接提供一个校验器方式
*	on(Predicate<? super T> validatorPredicate, String errorMsg)：断言不为真时，校验失败，并记录错误的errorMsg（内部会构造一个ValidationError）
*	on(Predicate<? super T> validatorPredicate, ValidationError error)：断言不为真时，校验失败，并记录提供的ValidationError。
*	on(Predicate<? super T> validatorPredicate, Stream<ValidationError> errors)：断言不为真时，校验失败，并记录提供的错误集合。

ValidationError构造方式如下：

	ValidationError.of("errorMessage")	// 错误消息（必填）
		.field("car")			// 错误字段（选填）
		.errorCode(123)			// 错误码（选填）
		.invalidValue(car);		// 错误值（选填）
		
其中errorMessage还可以带上params参数，供扩展使用，如：

	ValidationError.of("errorMessage:%d", messageNo); 
		
## 3.2 fail fast or fail over

当出现校验失败时，也就是出现了ValidationError，那么是继续还是直接退出呢？默认为使用failFast()方法，直接退出，如果你想继续完成所有校验，使用failOver()来skip掉。

	FunctionalValidator.checkFrom(car)
		.failFast()
		.on(Objects::nonNull, "car is null");
	FunctionalValidator.checkFrom(car)
		.failFast()
		.on(Objects::nonNull, "car is null");
		
## 3.3 onIf

类似于fluent-validator里的on()后面紧跟一个when()，functional-validator中把这个功能合并到了onIf()中，使用起来更加紧凑。onIf()在on()的几个方法基础上分别增加了一个断言参数，断言为真时才启用验证，否则skip验证。

	FunctionalValidator.checkFrom(car)
		.on(car -> car.getSeatCount() >= 2, "Seat count is not valid", car -> car != null);

## 3.4 对校验结果的处理

doValidate()方法进行验证后，返回FunctionalValidatorResult<T>类型的校验结果。FunctionalValidatorResult提供了丰富的函数式API对校验结果进行处理，之前示例中的fold()就是其中的一个，除此之外还有：

*	public final FunctionalValidatorResult<T> onResult(Consumer<ValidationResult> action)：对校验结果进行消费
*	public final FunctionalValidatorResult<T> onSuccess(Consumer<? super T> action)：校验成功时对校验对象进行消费
*	public final FunctionalValidatorResult<T> onFailure(Consumer<ValidationResult> action)：校验失败时对校验结果进行消费
*	public final FunctionalValidatorResult<T> onThrowable(Consumer<? super Throwable> action)：校验时出现异常时对异常进行消费
*	public final boolean isSuccess()：判断校验是否成功
*	public final &lt;U&gt; U foldIgnoreThrowable(Function<? super T, ? extends U> successMapper, Function<? super ValidationResult, ? extends U> failureMapper)：类似于fold()方法，但忽略对异常情况的规约，即调用该方法有可能会抛出异常，需要使用方自行处理或规避
*	public Try<ValidationResult> getResult()：获取校验结果，用Try数据结构包装校验过程中出现的异常。

Try数据结构参考javaslang实现，具体可参见[javaslang](http://www.javaslang.io/javaslang-docs/)中的Try。

ValidationResult数据结构定义如下：

	public final class ValidationResult {
		private List<ValidationError> errors = new LinkedList<>();	// 校验错误集合
		private boolean success = true;		// 校验是否成功，不可被外部应用直接修改
		private int globalErrorCode;		// 全局错误码（可通过addGlobalError()方法加入）
		private String globalErrorMessage;	// 全局错误信息（可通过addGlobalError()方法加入）
		private long timeElapsed;	// 校验耗时
	}
	
# 4 高级玩法

## 4.1 与JSR303规范最佳实现Hibernate Validator集成

和fluent-validator一样，functional-validator也站在巨人的肩膀上，把Hibernate Validator集成了进来，直接上代码：

	FunctionalValidator.checkFrom(car)
        .on(HibernateSupportedValidator.build().validator())
        .doValidate();
		
HibernateSupportedValidator依赖于javax.validation.Validator的实现，build()方法使用了Hibernate Validator官方提供的初始化javax.validation.Validator实现的方法，默认策略为failFast，也可以使用failOver策略的Validator:

	HibernateSupportedValidator.buildByFailOverValidator();

也可以自己提供validator：

	HibernateSupportedValidator.buildByValidator(myValidator).validator();
	
可以自定义从ConstraintViolation到ValidationError的转换器（如不指定，默认会提供如下的转换器）：

	HibernateSupportedValidator.build()
		.transformer(v -> ValidationError.of(v.getMessage())
            .field(v.getPropertyPath().toString())
            .invalidValue(v.getInvalidValue()))
		.validator();
		
也可以自定义Hibernate Validator校验出错时的错误码（默认为0）：

	HibernateSupportedValidator.build()
		.errorCode(234)
		.validator();
	
functional-validator对Hibernate Validator的集成主要依靠HibernateSupportedValidator类，整个类的实现都是基于函数式的，非常优雅、简洁，读者可通过阅读源码来体会下functional-validator通过函数来实现校验器的精妙之处。

## 4.2 Spring AOP集成

借助Spring AOP技术，Fuctional Validator实现了业务逻辑和验证逻辑的解耦，如下示例，业务代码中有个add方法需要校验参数：

	@Component
	public class Demo {
		
		@Valid(value = DemoValidator.class, method = "add", failFast = true, hibernateValidate = true, hibernateErrorCode = 0)
		public void add(Department department, Car car) {
			...
		}
	}
	
@Valid注解表示该方法的参数需要校验，其参数含义如下：

*	value：指明自定义校验器对应的Class，默认为Void.class（表明不需要自定义校验），以上的注解可简化为：@Valid(DemoValidator.class)，如果不需要自定义校验，仅仅使用集成的HibernateValidator进行校验，则注解形式可简化为：@Valid。
*	method：校验方法的别名，需要与实际校验器@ValidHandler注解上的value相匹配，默认为空字符串，表示与被注解的方法名相同。value相当于指定了校验器的类名，method相当于指定了校验器的方法名
*	failFast：校验出错是否立即返回
*	hibernateValidate：是否需要进行HibernateValidator的校验，如果为true，则先执行HibernateValidator的校验
*	hibernateErrorCode：Hibernate校验失败后需返回的错误码

接下来需要实现一个校验处理器，类名即为@Valid中指定的value：

	@Component
	public class DemoValidator {
		
		@ValidHandler("add")
		public void add(ValidationResult result, Department department, Car car) {
			if (car.getSeatCount() > 2) {
				result.addError(ValidationError.of("car seat count invalid"));
				return;
			}
			...
		}
	}
	
@ValidHandler注解表示该方法实现对参数的校验，是真正的校验逻辑，该注解参数只有1个：

*	value：校验方法的别名，默认为空，表示与被注解的方法名相同。

此时，每次调用Demo对象的add()方法前，就会执行DemoValidator的add()方法进行逻辑校验，如果校验失败，则会抛出ValidationException异常。如果你是Web工程，则可配合ExceptionHandler做对应的处理。

	
# 5 总结

首先要非常感谢[fluent-validator](https://github.com/neoremind/fluent-validator)项目对我创作的启发。本文从对fluent-validator框架实践中的痛点引出，展示了functional-validator更优雅、更易用的函数式API调用，以及对JSR303 – Bean Validation规范的集成，基本对functional-validator做了一个全面的介绍。

如果你的项目使用jdk8以上版本，那么希望functional-validator框架可以帮助你更好的做业务逻辑验证，同时也希望你能对该框架提出宝贵的意见或贡献。

最新更新，请参考[github](https://github.com/javadeep/common-functional)