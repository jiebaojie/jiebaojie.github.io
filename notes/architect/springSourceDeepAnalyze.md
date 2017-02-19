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