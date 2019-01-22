# 自定义拦截器
## 1、创建拦截器，实现HandlerInterceptor接口
这里创建两个拦截器，FirstInterceptor 和 SecondInterceptor<br>
FirstInterceptor
```
package com.atguigu.springmvc.interceptors;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

public class FirstInterceptor implements HandlerInterceptor{

	/**
	 * 渲染视图之后被调用
	 * 
	 * 释放资源
	 */
	@Override
	public void afterCompletion(HttpServletRequest arg0, HttpServletResponse arg1, Object arg2, Exception arg3)
			throws Exception {
		System.out.println("[FirstInterceptor] afterCompletion");
		
	}

	/**
	 * 调用目标方法之后，渲染视图之前被调用
	 * 
	 * 可以对请求域中的属性或视图做出修改
	 */
	@Override
	public void postHandle(HttpServletRequest arg0, HttpServletResponse arg1, Object arg2, ModelAndView arg3)
			throws Exception {
		System.out.println("[FirstInterceptor] postHandle");
		
	}

	/**
	 * 该方法在目标方法之前被调用，
	 * 若返回值为true，则继续调用后续的拦截器和目标方法
	 * 若返回值为false，则不会再调用后续的拦截器和目标方法。
	 * 
	 * 可以考虑做权限，日志，事务等
	 */
	@Override
	public boolean preHandle(HttpServletRequest arg0, HttpServletResponse arg1, Object arg2) throws Exception {
		System.out.println("[FirstInterceptor] preHandle");
		return true;
	}
	
}

```
SecondInterceptor
```
package com.atguigu.springmvc.interceptors;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

public class SecondInterceptor implements HandlerInterceptor{

	/**
	 * 渲染视图之后被调用
	 * 
	 * 释放资源
	 */
	@Override
	public void afterCompletion(HttpServletRequest arg0, HttpServletResponse arg1, Object arg2, Exception arg3)
			throws Exception {
		System.out.println("[SecondInterceptor] afterCompletion");
		
	}

	/**
	 * 调用目标方法之后，渲染视图之前被调用
	 * 
	 * 可以对请求域中的属性或视图做出修改
	 */
	@Override
	public void postHandle(HttpServletRequest arg0, HttpServletResponse arg1, Object arg2, ModelAndView arg3)
			throws Exception {
		System.out.println("[SecondInterceptor] postHandle");
		
	}

	/**
	 * 该方法在目标方法之前被调用，
	 * 若返回值为true，则继续调用后续的拦截器和目标方法
	 * 若返回值为false，则不会再调用后续的拦截器和目标方法。
	 * 
	 * 可以考虑做权限，日志，事务等
	 */
	@Override
	public boolean preHandle(HttpServletRequest arg0, HttpServletResponse arg1, Object arg2) throws Exception {
		System.out.println("[SecondInterceptor] preHandle");
		return true;
	}
	
}
```
![图片无法加载](https://github.com/Ywfy/Learning-summary-for-SpringMVC/blob/master/interceptor/lz1.PNG)<br>
<br>

## 2、配置拦截器
```
<mvc:interceptors>
		
   <!-- 配置自定义拦截器 -->
   <bean class="com.atguigu.springmvc.interceptors.FirstInterceptor"></bean>
    
   <!-- 配置拦截器(不)作用的路径 -->
   <mvc:interceptor>
      <!-- <mvc:exclude-mapping path=""/> -->
      <!-- 拦截器只作用于/emps -->
      <mvc:mapping path="/emps"/>
      <bean class="com.atguigu.springmvc.interceptors.SecondInterceptor"></bean>
   </mvc:interceptor>
		
</mvc:interceptors>
```

根据拦截器的配置，当访问/emps时，两个拦截器都会生效，这时执行顺序是怎样的呢？
我这边运行后的结果如下:
![无法加载图片](https://github.com/Ywfy/Learning-summary-for-SpringMVC/blob/master/interceptor/lzr.PNG)<br>
答案是：
```
preHandle 按拦截器配置的顺序执行

postHandle 按配置的反序执行

afterCompletion 按配置的反序执行
```
我们先配置的FirstInterceptor，后配置的SecondInterceptor<br>
![无法加载图片](https://github.com/Ywfy/Learning-summary-for-SpringMVC/blob/master/interceptor/dlz.PNG)<br>
preHandle有成功执行完成的，就必须执行对应拦截器的afterCompletion<br>


