# 网页国际化
## 1、在页面上能够根据浏览器语言设置的情况对文本(不是内容),时间，数值进行本地化处理
### 步骤1：创建i18n配置文件
i18n.properties
```
i18n.user=\u7528\u6237\u540D
i18n.password=\u5BC6\u7801
```
i18n_en_US.properties
```
i18n.user=User
i18n.password=Password
```
i18n_zh_CN.properties
```
i18n.user=\u7528\u6237\u540D
i18n.password=\u5BC6\u7801
```
注意上方右边的那些看着像乱码的东西，其实我们打进去的是中文，只是他自动显示成这样子<br>
<br>

### 步骤2：在springmvc配置文件 配置国际化资源文件标签'
```
<!-- 配置国际化资源文件 -->
	<bean id="messageSource"
		class="org.springframework.context.support.ResourceBundleMessageSource">
		<property name="basename" value="i18n"></property>	
	</bean>
```

以上就实现了网页显示国际化<br>
<br>

## 2、可以在bean中获取国际化资源文件Locale 对应的消息
### 步骤1：创建jsp页面
#### 修改index.jsp
```
添加超链接
 <a href="i18n">I18N PAGE</a>
```

#### 在/WEB-INF/views目录下创建i18n.jsp
```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ taglib prefix="fmt" uri="http://java.sun.com/jstl/fmt" %>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
	
	<fmt:message key="i18n.user"></fmt:message>
	
</body>
</html>
```
### 步骤2：编写控制器处理方法
```
@RequestMapping("/i18n")
	public String testi18n(Locale locale) {
		String val = messageSource.getMessage("i18n.user", null, locale);
		System.out.println(val);
		return "i18n";
	}
```
<br>
运行后，通过切换浏览器的语言能够让eclipse控制台输出不同的内容<br>
<br>

## 3、通过超链接切换Locale,而不再依赖于浏览器的语言设置情况
![无法加载图片](https://github.com/Ywfy/Learning-summary-for-SpringMVC/blob/master/Employee/img/yl.png)<br>
通过配置LocalResolver和 LocaleChangeInterceptor就可以实现
```
<!-- 配置SessionLocalResolver用于将Locale对象存储于Session中供后续使用 -->
<!-- 该处id名只能为localeResolver,否则运行报错 -->
<bean id="localeResolver"
	class="org.springframework.web.servlet.i18n.SessionLocaleResolver">
</bean>
	
<mvc:interceptors>
	<!-- 配置LocaleChangeInterceptor主要用于获取请求中的locale信息，将其转为Locale对像，获取LocaleResolver对象 -->
	<bean class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor"></bean>
</mvc:interceptors>
```
修改i18n.jsp
```
<fmt:message key="i18n.user"></fmt:message>
	
<br><br>
<a href="i18n?locale=zh_CH">中文</a>
	
<br><br>
<a href="i18n?locale=en_US">英文</a>
```
运行<br>
通过点击"中文"或者"英文"超链接就可以切换页面的语言<br>
<br>


