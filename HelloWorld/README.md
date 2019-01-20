# HelloWorld

## 1、加入jar包
新建Dynamic Web Project，输入项目名，点击两次next，勾上Generate web.xml deployment descriptor，点击finish<br>
在/WEB-INF/lib目录下放入以下jar包
```
commons-logging-1.1.3.jar
spring-aop-4.0.0.RELEASE.jar
spring-beans-4.0.0.RELEASE.jar
spring-context-4.0.0.RELEASE.jar
spring-core-4.0.0.RELEASE.jar
spring-expression-4.0.0.RELEASE.jar
spring-web-4.0.0.RELEASE.jar
spring-webmvc-4.0.0.RELEASE.jar
taglibs-standard-compat-1.2.5.jar
taglibs-standard-impl-1.2.5.jar
taglibs-standard-jstlel-1.2.5.jar
taglibs-standard-spec-1.2.5.jar
```

## 2、在 web.xml 中配置 DispatcherServlet
```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://xmlns.jcp.org/xml/ns/javaee"
	xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
	id="WebApp_ID" version="3.1">
  
	<!-- 配置DispatcherServlet -->
	<!-- The front controller of this Spring Web application, responsible for handling all application requests -->
	<servlet>
		<servlet-name>springDispatcherServlet</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<!-- 配置DispatcherServlet的一个初始化参数：作用是配置SpringMVC配置文件的位置和名称 -->
		<!-- 
		实际上也可以不通过contextConfigLocation来配置SpringMVC的配置文件，而使用默认的 。
		默认的配置文件为:/WEB-INF/<servlet-name>-servlet.xml
		-->
		<!-- 
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>classpath:springmvc.xml</param-value>
		</init-param> 
		-->
		<load-on-startup>1</load-on-startup>
	</servlet>

	<!-- Map all requests to the DispatcherServlet for handling -->
	<servlet-mapping>
		<servlet-name>springDispatcherServlet</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>
</web-app>
```

## 3、创建 Spring MVC 配置文件
在web.xml指定的位置下，因为先前我们没有指定，所以是在默认位置/WEB-INF下创建springDispatcherServlet-servlet.xml
```
  <!-- 配置扫描包 -->
	<context:component-scan base-package="com.atguigu.springmvc"></context:component-scan>

	<!-- 配置视图解析器:如何把handler方法返回值解析为实际的物理视图 -->
	<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<property name="prefix" value="/WEB-INF/views/"></property>
		<property name="suffix" value=".jsp"></property>
	</bean>
```

## 4、创建请求处理器类
```
package com.atguigu.springmvc.handlers;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class HelloWorld {
	
	/**
	 * 1、使用@RequestMapping注解来映射请求的url
	 * 即当进入/helloworld后就会执行hello()方法
	 * 2、返回值会通过视图解析器解析为实际的物理视图，对于InternalResourceViewResolver视图解析器，会做
	 * 	如下的解析：
	 * 	通过	prefix + returnVal + suffix 方式得到实际的物理视图，然后做转发操作
	 * /WEB-INF/views/success.jsp
	 * @return
	 */
	@RequestMapping("/helloworld")
	public String hello() {
		System.out.println("hello World!");
		return "success";
	}
}
```

## 5、编写视图
index.jsp:在\<body>内写一个超链接<br>
```
<a href="helloworld">Hello World</a>
```


success.jsp:(位于/WEB-INF/views/),在\<body>内写一个文本标签
```
<h4>Success Page</h4>
```
