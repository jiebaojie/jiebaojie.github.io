---
layout: post
notes: true
subtitle: "Spring源码深度解析"
comments: false
author: "郝佳"
date: 2017-02-19 00:00:00

---

![](/img/notes/architect/springSourceDeepAnalyze/spring_source_deep_analyze.jpg)

*   目录
{:toc }

### 11.4.5 缓存处理

Last-Modified缓存机制：

1.	在客户端第一次输入URL时，服务器端会返回内容和状态码200，表示请求成功，同时会添加一个"Last-Modified"的响应头，表示此文件在服务器上的最后更新时间。
2.	客户端第二次请求此URL时，客户端会向服务器发送请求头"If-Modified-Since"，询问服务器该时间之后当前请求内容是否有被修改过，如果服务器端的内容没有变化，则自动返回HTTP 304状态码（只要响应头，内容为空，这样就节省了网络带宽）。

Spring提供了对Last-Modified机制的支持，只需要实现LastModified接口，接口的getLastModified方法保证当内容发生改变时返回最新的修改时间即可。

Spring判断是否过期，通过判断请求的"If-Modified-Since"是否大于等于当前的getLastModified方法的时间戳，如果是，则认为没有修改。

### 11.4.6 HandlerInterceptor的处理

Servlet API定义的servlet过滤器可以在servlet处理每个Web请求的前后分别对它进行前置处理和后置处理。

SpringMVC允许你通过处理器拦截Web请求，进行前置处理和后置处理。处理器拦截是在Spring的Web应用程序上下文中配置的，因此它们可以利用各种容器特性，并引用容器中声明的任何bean。处理拦截是针对特殊的处理程序映射进行注册的，因此它只拦截通过这些处理程序映射的请求。每个处理拦截都必须实现HandlerInterceptor接口，它包含三个需要你实现的回调方法：preHandle()、postHandle()和afterCompletion()。第一个和第二个方法分别是在处理程序处理请求之前和之后被调用的。第二个方法还允许访问返回的ModelAndView对象，因此可以在它里面操作模型属性。最后一个方法是在所有请求处理完成之后被调用的（如视图呈现之后）。

	public class MyTestInterceptor implements HandlerInterceptor {
		
		@Override
		public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object hendler) throws Exception {
			long startTime = System.currentTimeMillis();
			request.setAttribute("startTime", startTime);
			return true;
		}
		
		@Override
		public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
			long startTime = (Long) request.getAttribute("startTime");
			request.removeAttribute("startTime");
			long endTime = System.currentTimeMillis();
			modelAndView.addObject("handlingTime", endTime - startTime);
		}
		
		@Override
		public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
			
		}
	}
	
在这个拦截器的preHandler()方法中，你记录了起始时间，并将它保存到请求属性中。这个方法应该返回true，允许DispatcherServlet继续处理请求。否则，DispatcherServlet会认为这个方法已经处理了请求，直接将响应返回给用户。