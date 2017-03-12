---
layout: post
notes: true
subtitle: "【技术博客】正确实现用spring扫描自定义的annotation"
comments: false
author: "hengyunabc"
date: 2017-02-19 00:00:00

---


原文：[http://blog.csdn.net/hengyunabc/article/details/51289327](http://blog.csdn.net/hengyunabc/article/details/51289327)

*   目录
{:toc }

# 背景

在使用spring时，有时候有会有一些自定义annotation的需求，比如一些Listener的回调函数。

比如：

	@Service
	public class MyService {
		@MyListener
		public void onMessage(Message msg) {
		}
	}

# BeanPostProcessor接口

实现BeanPostProcessor接口才是最合理的办法（可参考spring jms里的@JmsListener的实现）

	public interface BeanPostProcessor {
	
		Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;
		
		Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;
	}
	
所有的bean在创建完之后，都会回调postProcessAfterInitialization函数，这时就可以确定bean是已经创建好的了。

所以扫描自定义的annotation的代码大概是这个样子的：

	public class MyListenerProcessor implements BeanPostProcessor {
		@Override
		public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
			return bean;
		}
		
		@Override
		public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
			Method[] methods = ReflectionUtils.getAllDeclaredMethods(bean.getClass());
			if (methods != null) {
				for (Method method : methods) {
					MyListener myListener = AnnotationUtils.findAnnotation(method, MyListener.class);
					// process
				}
			}
			return bean;
		}
	}
	
# SmartInitializingSingleton接口

SmartInitializingSingleton 这个接口也比较有意思。就是当所有的singleton的bean都初始化完了之后才会回调这个接口。不过要注意是 4.1 之后才出现的接口。

	public interface SmartInitializingSingleton {
		
		void afterSingletonsInstantiated();
	}
