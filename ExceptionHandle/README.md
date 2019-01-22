# 异常处理
我们先制造一个异常<br>
## 1、<mvc:annotation-driven></mvc:annotation-driven> 强烈建议开发就直接写上
## 2、修改index.jsp
```
<a href="testExceptionHandlerExceptionRosolver?i=10">Test ExceptionHandlerExceptionRosolver</a>
```

## 3、创建处理器方法
```
  private static final String SUCCESS = "success";

  @RequestMapping("/testExceptionHandlerExceptionRosolver")
	public String testExceptionHandlerExceptionRosolver(
			@RequestParam("i") Integer i) {
		System.out.println("result: " + (10 / i));
		return SUCCESS;
	}
```
<strong>success.jsp页面的创建就不说了，其他多余的配置若不清楚请翻看前面文章</strong><br>

很明显，运行后，若我们将i=0的话就会出现异常<br>

## 4、创建异常处理器
在controller下创建
```
  //Exception类型的参数,该参数即对应发生的异常对象。
  @ExceptionHandler({ArithmeticException.class})
	public ModelAndView handleArithmeticException(Exception ex) {
		System.out.println("出异常了: " + ex);
		return error;
```

## 5、创建error.jsp
```
<body>
	<h4>Error Page</h4>
</body>
```
我这里只标明body部分，其他部分不用改<br>

显然，这时当我们将i=0时，就会跳转到error.jsp页面，并控制台会打印信息<br>

## 6、将异常信息打印到页面上
想要将异常信息打印到页面上，也许我们会首先想到用map，但是这里不行，用map的话运行会报错，有兴趣的小伙伴可以试下。<br>
@ExceptionHandler 方法的入参中不能传入Map，若希望把异常信息传到页面上，需要使用ModelAndView作为返回值<br>
```
  @ExceptionHandler({ArithmeticException.class})
	public ModelAndView handleArithmeticException(Exception ex) {
		System.out.println("出异常了: " + ex);
		ModelAndView mv = new ModelAndView("error");
		mv.addObject("exception", ex);
		return mv;
	}
```
修改error.jsp
```
<body>
	<h4>Error Page</h4>
	
	${requestScope.ex }
</body>
```
此时运行，异常成功地被打印到页面上<br>

## 7、@ExceptionHandler 方法标记的异常有优先级的问题
复制原来的处理方法，只是标记的异常改为了RuntimeException.class
```
  @ExceptionHandler({RuntimeException.class})
	public ModelAndView handleArithmeticException2(Exception ex) {
		System.out.println("[出异常了]: " + ex);
		ModelAndView mv = new ModelAndView("error");
		mv.addObject("exception", ex);
		return mv;
	}
```
此时，我们运行后，他会执行哪个异常处理方法呢？<br>
根据异常的类型来判断，标记的异常类型越接近实际抛出的异常，就执行哪个方法！<br>
而显然，我们i=0的话最后抛出的就是ArithmeticException，所以会执行原来的方法<br>

## 8、全局异常处理方法
原来的异常处理方法都是写在controller下的，这样就只能处理当前controller抛出的异常<br>
若我们想要实现可以处理全局异常的话，SpringMVC存在以下机制：<br>
<strong>如果在当前Handler中找不到@ExceptionHandler方法来处理当前方法出现的异常，则将去@ControllerAdvice标记的类中查找@ExceptionHandler标记的方法来处理异常</strong><br><br>

示例：
```
package com.atguigu.springmvc.test;

import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.servlet.ModelAndView;

@ControllerAdvice
public class SpringMVCTestExceptionHandler {
	
	@ExceptionHandler({ArithmeticException.class})
	public ModelAndView handlerArithmeticException(Exception ex) {
		System.out.println("------->出异常了: " + ex);
		ModelAndView mv = new ModelAndView("error");
		mv.addObject("exception", ex);
		return mv;
	}
}
```
此时我们将原来的两个异常处理方法注释掉，运行后，i=0就会执行上面这段异常处理代码<br>
<br>

## 9、@ResponseStatus设置异常页面响应信息
### 1)、创建一个异常
```
package com.atguigu.springmvc.test;

import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ResponseStatus;

public class UsernameNotMatchPasswordException extends RuntimeException{

	/**
	 * 
	 */
	private static final long serialVersionUID = 1L;

}
```
### 2)、创建处理器代码
```
	@RequestMapping("/testResponseStatusExceptionResolver")
	public String testResponseStatusExceptionResolver(
			@RequestParam("i") int i) {
		if(i == 13) {
			throw new UsernameNotMatchPasswordException();
		}
		System.out.println("testResponseStatusExceptionResolver...");
		return "success";
	}
```

### 3)、创建超链接请求
修改index.jsp
```
<a href="testResponseStatusExceptionResolver?i=10">Test ResponseStatusExceptionResolver</a>
```
此时，若我们运行，将i=13，就会抛出UsernameNotMatchPasswordException

### 4）、添加@ResponseStatus
#### 添加在异常类上
```
@ResponseStatus(value=HttpStatus.FORBIDDEN, reason="用户名和密码不匹配")
public class UsernameNotMatchPasswordException extends RuntimeException{
```
value代表异常页面状态码<br>

运行，令i=13<br>
![无法加载图片](https://github.com/Ywfy/Learning-summary-for-SpringMVC/blob/master/ExceptionHandle/rs1.PNG)<br>

#### 添加在方法上
```
  @ResponseStatus(reason="测试", value=HttpStatus.NOT_FOUND)
	@RequestMapping("/testResponseStatusExceptionResolver")
	public String testResponseStatusExceptionResolver(
			@RequestParam("i") int i) {
      ......
```
运行<br>
<strong>注意：此时i=10，正常来说是能够正常运行</strong>
![无法加载图片](https://github.com/Ywfy/Learning-summary-for-SpringMVC/blob/master/ExceptionHandle/rs2.PNG)<br>
控制台页面有正常的打印信息<br>
@ResponseStatus添加到方法上，会强制将返回页面转为状态码对应的页面。<br>
<br>

扩展：同时标注在UsernameNotMatchPasswordException和方法上，若i=13，最终的返回页面是异常对应的页面<br>
<br>

## SimpleMappingExceptionResolver
如果希望对所有异常进行统一处理，可以使用SimpleMappingExceptionResolver，它将异常类名映射为视图名，即发生异常时使用对应的视图报告异常<br>
### 1)、修改index.jsp
```
<a href="testSimpleMappingExceptionResolver?i=5">Test SimpleMappingExceptionResolver</a>
```
### 2)、编写处理器方法
```
  @RequestMapping("/testSimpleMappingExceptionResolver")
	public String testSimpleMappingExceptionResolver(
			@RequestParam("i") Integer i) {
		String[] vals = new String[10];
		System.out.println(vals[i]);
		return SUCCESS;
	}
```
显然，这里若传入的i>=10，就会抛出异常<br>

### 3)、配置使用SimpleMappingExceptionResolver来映射异常
```
<bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
		<property name="exceptionMappings">
			<props>
				<prop key="java.lang.ArrayIndexOutOfBoundsException">error</prop>
			</props>
		</property>
	</bean>
```
运行，传入i=21，此时就会跳转到error页面，此时你会发现一个惊奇的页面<br>
![无法加载图片](https://github.com/Ywfy/Learning-summary-for-SpringMVC/blob/master/ExceptionHandle/sm2.PNG)<br>
SimpleMappingExceptionResolver自动将抛出的异常放入了Request域里面，我们可以在返回页面进行调用显示<br>



