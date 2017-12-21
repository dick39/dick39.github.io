# spring boot下对filter、interceptor、aop做比较记录

为加深对web的过滤器、拦截器、切面的了解，使用spring-boot快速搭建了一个包含多个拦截的web。x需求是统计接口耗时，搭建详情如下：

1. Filter:

   创建TimeFilter实现Filter接口

   ```java
       @Override
   	public void init(FilterConfig filterConfig) throws ServletException {
   		System.out.println("time filter init");
   	}

   	@Override
   	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
   			throws IOException, ServletException {
   		System.out.println("time filter start :" + ((HttpServletRequest)request).getRequestURL());
   		long start = System.currentTimeMillis();
   		chain.doFilter(request, response);
   		System.out.println("time filter 耗时：" + (System.currentTimeMillis() - start));
   		System.out.println("time filter end");
   	}

   	@Override
   	public void destroy() {
   		System.out.println("time filter destroy");
   	}
   ```

2. Interceptor:

   创建TimeInterceptor实现HandlerInterceptor接口并注册

   ~~~java
   	@Override
   	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
   			throws Exception {
   		System.out.println("TimeInterceptor preHandle");
   		System.out.println(((HandlerMethod)handler).getClass().getName() + "的" + ((HandlerMethod)handler).getMethod().getName());
   		request.setAttribute("startTime", System.currentTimeMillis());
   		return true;
   	}

   	@Override
   	public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
   			ModelAndView modelAndView) throws Exception {
   		System.out.println("TimeInterceptor postHandle");
   	}

   	@Override
   	public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)
   			throws Exception {
   		System.out.println("TimeInterceptor 耗时:" + (System.currentTimeMillis() - (long)request.getAttribute("startTime")));
   		System.out.println("ex is : " + ex);
   		System.out.println("TimeInterceptor afterCompletion");
   	}
   ~~~

3. AOP:

   创建TimeAspect类环绕式的切面

   ~~~java
   @Around("execution(* com.sheng.controller.UserController.*(..))")
   	public void timeAspect(ProceedingJoinPoint pjp) throws Throwable{
   		System.out.println("time aspect start");
   		Object[] args = pjp.getArgs();
   		System.out.println("time aspect arg is " + args[0]);
   		args[0] = "2";
   		long start = System.currentTimeMillis();
   		pjp.proceed(args);
   		System.out.println("time aspect 耗时:" + (System.currentTimeMillis() - start));
   		System.out.println("time aspect end");
   	}
   ~~~



### 调用正常返回接口

控制台输出为：

~~~txt
time filter start :http://localhost:8080/user/1
TimeInterceptor preHandle
org.springframework.web.method.HandlerMethod的getUserInfo
time aspect start
time aspect arg is 1
time aspect 耗时:4
time aspect end
TimeInterceptor postHandle
TimeInterceptor 耗时:69
ex is : null
TimeInterceptor afterCompletion
time filter 耗时：84
time filter end
~~~

### 调用异常返回接口

控制台输出为：

~~~txt
time filter start :http://localhost:8080/user/getException/1
TimeInterceptor preHandle
org.springframework.web.method.HandlerMethod的getException
time aspect start
time aspect arg is 1
TimeInterceptor 耗时:24
ex is : com.sheng.exception.NotExistException: not exist
TimeInterceptor afterCompletion
com.sheng.exception.NotExistException: not exist......
~~~

在注册了ControllerAdvice之后，控制台输出为：

~~~txt
time filter start :http://localhost:8080/user/getException/1
TimeInterceptor preHandle
org.springframework.web.method.HandlerMethod的getException
time aspect start
time aspect arg is 1
TimeInterceptor 耗时:87
ex is : null
TimeInterceptor afterCompletion
time filter 耗时：102
time filter end
~~~



### 思考

以上返回日志就结果来说并无意外，并且可以得出网上的图：

![拦截图.png](../../img/%E6%8B%A6%E6%88%AA.png)

发现抛出异常时，在注册了ControllerAdvice的情况下，拦截器并不会捕获异常，但是postHandle方法并不会调用，表明拦截器获取到或者未调用postHandle方法。

进入DispatcherServlet源码可以观察到：

~~~java
try {
  if (!mappedHandler.applyPreHandle(processedRequest, response)) {
    return;
  }
  // Actually invoke the handler.
  mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
  if (asyncManager.isConcurrentHandlingStarted()) {
    return;
  }
  applyDefaultViewName(processedRequest, mv);

  mappedHandler.applyPostHandle(processedRequest, response, mv);
}
catch (Exception ex) {
  dispatchException = ex;
}
catch (Throwable err) {
  // As of 4.3, we're processing Errors thrown from handler methods as well,
  // making them available for @ExceptionHandler methods and other scenarios.
  dispatchException = new NestedServletException("Handler dispatch failed", err);
}
processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
~~~

第二行mappedHandler.applyPreHandle(..)调用拦截器的preHandle方法

第十四行mappedHandler.applyPostHandle(..)调用拦截器的postHandle方法

最后一行的processDispatchResult(..)调用拦截器的afterCompletion方法

可以看到当拦截器的preHandle方法返回false，直接跳出整个方法；当handle方法执行抛出异常时，会直接跳过applyPostHandle方法，除非aop处理该异常。