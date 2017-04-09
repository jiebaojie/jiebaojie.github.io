---
layout: post
notes: true
subtitle: "【技术博客】Java核心技术点之动态代理"
comments: false
author: "absfree"
date: 2017-04-09 00:00:00

---


原文：[http://www.cnblogs.com/absfree/p/5392639.html](http://www.cnblogs.com/absfree/p/5392639.html)

*   目录
{:toc }

# 一、概述

## 1. 什么是代理

通过使用代理，通常有两个优点：

*	优点一：可以隐藏委托类的实现；
*	优点二：可以实现客户与委托类间的解耦，在不修改委托类代码的情况下能够做一些额外的处理。

## 2. 静态代理

若代理类在程序运行前就已经存在，那么这种代理方式被成为静态代理，这种情况下的代理类通常都是我们在Java代码中定义的。 通常情况下，静态代理中的代理类和委托类会实现同一接口或是派生自相同的父类。

	public interface Sell {
		void sell();
		void ad();
	}

委托类和代理类都实现了Sell接口，Vendor类的定义如下：

	public class Vendor implements Sell {
		public void sell() {
			System.out.println("In sell method");
		}
		public void ad() {
			System,out.println("ad method")
		}
	}
	
代理类BusinessAgent的定义如下：

	public class BusinessAgent implements Sell {
		private Vendor mVendor;
    
		public BusinessAgent(Vendor vendor) {
			mVendor = vendor;
		}
     
		public void sell() { mVendor.sell(); }
		public void ad() { mVendor.ad(); }
	}
	
从BusinessAgent类的定义我们可以了解到，静态代理可以通过聚合来实现，让代理类持有一个委托类的引用即可。

下面我们考虑一下这个需求：给Vendor类增加一个过滤功能，只卖货给大学生。通过静态代理，我们无需修改Vendor类的代码就可以实现，只需在BusinessAgent类中的sell方法中添加一个判断即可如下所示：

	public class BusinessAgent implements Sell {
		...
		public void sell() {
			if (isCollegeStudent()) {
				vendor.sell();
			}
		}
		...
	}
	
这对应着我们上面提到的使用代理的第二个优点：可以实现客户与委托类间的解耦，在不修改委托类代码的情况下能够做一些额外的处理。静态代理的局限在于运行前必须编写好代理类，下面我们重点来介绍下运行时生成代理类的动态代理方式。

# 二、动态代理

## 1. 什么是动态代理

代理类在程序运行时创建的代理方式被成为**动态代理**。也就是说，这种情况下，代理类并不是在Java代码中定义的，而是在运行时根据我们在Java代码中的“指示”动态生成的。相比于静态代理，**动态代理的优势在于可以很方便的对代理类的函数进行统一的处理，而不用修改每个代理类的函数。**

现在，假设我们要实现这样一个需求：在执行委托类中的方法之前输出“before”，在执行完毕后输出“after”。我们还是以上面例子中的Vendor类作为委托类，BusinessAgent类作为代理类来进行介绍。首先我们来使用静态代理来实现这一需求，相关代码如下：

	public class BusinessAgent implements Sell {
		private Vendor mVendor;
		
		public BusinessAgent(Vendor vendor) {
			this.mVendor = vendor;
		}

		public void sell() {
			System.out.println("before");
			mVendor.sell();
			System.out.println("after");
		}

		public void ad() {
			System.out.println("before");
			mVendor.ad();
			System.out.println("after");
		}
	}
	
从以上代码中我们可以了解到，通过静态代理实现我们的需求需要我们在每个方法中都添加相应的逻辑，这里只存在两个方法所以工作量还不算大，假如Sell接口中包含上百个方法呢？这时候使用静态代理就会编写许多冗余代码。通过使用动态代理，我们可以做一个“统一指示”，从而对所有代理类的方法进行统一处理，而不用逐一修改每个方法。

## 2. 使用动态代理

### （1）InvocationHandler接口

在使用动态代理时，我们需要定义一个位于代理类与委托类之间的中介类，这个中介类被要求实现InvocationHandler接口，这个接口的定义如下：

	public interface InvocationHandler {
		Object invoke(Object proxy, Method method, Object[] args);
	}
	
### （2）委托类的定义

动态代理方式下，要求委托类必须实现某个接口，这里我们实现的是Sell接口。委托类Vendor类的定义如下：

	public class Vendor implements Sell {
		public void sell() {
			System.out.println("In sell method");
		}
		public void ad() {
			System,out.println("ad method")
		}
	}
	
### （3）中介类

中介类必须实现InvocationHandler接口，作为调用处理器”拦截“对代理类方法的调用。

	public class DynamicProxy implements InvocationHandler {
		private Object obj; //obj为委托类对象；
      
		public DynamicProxy(Object obj) {
			this.obj = obj;
		}
  
		@Override
		public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
			System.out.println("before");
			Object result = method.invoke(obj, args);
			System.out.println("after");
			return result;
		}
	}
	
### （4）动态生成代理类

动态生成代理类的相关代码如下：

	public class Main {
		public static void main(String[] args) {
			//创建中介类实例
			DynamicProxy  inter = new DynamicProxy(new Vendor());
			//加上这句将会产生一个$Proxy0.class文件，这个文件即为动态生成的代理类文件
			System.getProperties().put("sun.misc.ProxyGenerator.saveGeneratedFiles","true"); 
			//获取代理类实例sell
			Sell sell = (Sell)(Proxy.newProxyInstance(Sell.class.getClassLoader(),
			new Class[] {Sell.class}, inter));
			//通过代理类对象调用代理类方法，实际上会转到invoke方法调用
			sell.sell();
			sell.ad();
		}
	}
	
在以上代码中，我们调用Proxy类的newProxyInstance方法来获取一个代理类实例。这个代理类实现了我们指定的接口并且会把方法调用分发到指定的调用处理器。这个方法的声明如下：

	public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h) throws IllegalArgumentException
	
方法的三个参数含义分别如下：

*	loader：定义了代理类的ClassLoder；
*	interfaces：代理类实现的接口列表
*	h：调用处理器，也就是我们上面定义的实现了InvocationHandler接口的类实例

原理：首先通过newProxyInstance方法获取代理类实例，而后我们便可以通过这个代理类实例调用代理类的方法，对代理类的方法的调用实际上都会调用中介类（调用处理器）的invoke方法，在invoke方法中我们调用委托类的相应方法，并且可以添加自己的处理逻辑。

## 3. 动态代理类的源码分析

阅读newProxyInstance方法的源码即可，这个方法的逻辑也没有复杂的地方。

