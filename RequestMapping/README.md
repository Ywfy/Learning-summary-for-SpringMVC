# RequestMapping
* Spring MVC 使用 @RequestMapping 注解为控制器指定可以处理哪些 URL 请求
* @RequestMapping除了修饰方法，还可以修饰类
	 * 类定义处：提供初步的请求映射信息，相对于WEB应用的根目录
	 * 方法处：提供进一步的细分映射信息，相对于类定义处的URL。若在类没有定义，则相对于WEB应用的根目录
示例:
```
@RequestMapping("/springmvc")
@Controller
public class SpringMVCTest {
	
	private static final String SUCCESS = "success";

	@RequestMapping("/testRequestMapping")
	public String testRequestMapping() {
		System.out.println("testRequestMapping");
		return SUCCESS;
	}
  
  ...
  
  这种情况下访问是/springmvc/testRequestMapping才会调用testRequestMapping()方法
  注意：上面的地址是相对于 WEB 容器部署的根路径的
```
* DispatcherServlet 截获请求后，就通过控制器上@RequestMapping 提供的映射信息确定请求所对应的处理方法
<br>

## 标准HTTP请求报头<br>
![图片无法加载](https://github.com/Ywfy/Learning-summary-for-SpringMVC/blob/master/RequestMapping/%E6%8D%95%E8%8E%B7.PNG)

## 2、RequestMapping属性(设置映射请求)
	 * value 请求url
	 * method 请求方法
	 * params 请求参数
	 * heads 请求头
	 他们之间是与的关系
   
```
  @RequestMapping("/testRequestMapping")
	public String testRequestMapping() {
		System.out.println("testRequestMapping");
		return SUCCESS;
	}
	
	/**
	 * 常用:使用method属性来指定请求方式:
	 * 	1)method=RequestMethod.POST
	 *  2)method=RequestMethod.GET
	 */
	@RequestMapping(value="/testMethod", method=RequestMethod.POST)
	public String testMethod() {
		System.out.println("testMethod");
		return SUCCESS;
	}
	
	/**
	 * 了解:可以使用params 和 headers 来更加精确地映射请求， params 和headers 支持简单的表达式。
	 * @return
	 */
	@RequestMapping(value="/testParamsAndHeaders", 
			params= {"username","age!=10"}, headers= {"Accept-Language=en-US,zh;q=0.9"})
	public String TestParamsAndHeaders() {
		System.out.println("testParamsAndHeaders");
		return SUCCESS;
	}
	
```
<br>


## @requestMapping支持Ant风格的URL

Ant风格资源地址支持3种匹配符:<br>
	 1） \?匹配文件名中的一个字符<br>
	 2） \*匹配文件名中的任意字符<br>
	 3）\**匹配多层路径<br>
```
- /user/*/createUser: 匹配
/user/aaa/createUser、/user/bbb/createUser 等 URL

– /user/**/createUser: 匹配
/user/createUser、/user/aaa/bbb/createUser 等 URL

– /user/createUser??: 匹配
/user/createUseraa、/user/createUserbb 等 URL
```
<br>

## @PathVariable
```
/**
	 * @PathVariable 可以来映射URL 中的占位符到目标方法的参数中。
	 */
	@RequestMapping("testPathVariable/{id}")
	public String testPathVariable(@PathVariable("id") Integer id) {
		System.out.println("testPathVariable: " + id);
		return SUCCESS;
	}
	
	@RequestMapping(value="/testRest/{id}", method=RequestMethod.GET)
	public String testRest(@PathVariable("id") Integer id) {
		System.out.println("testRest GET: " + id);
		return SUCCESS;
	}
```
<br>

## 映射请求参数 & 请求头
### @RequestParam
```

<a href="springmvc/testRequestParam?username=atguigu&age=11">Test RequestParam</a>


	/**
	 * @RequestParam 来映射请求参数
	 * value 值即请求参数的参数名
	 * required 该参数是否必须，默认为true
	 * defaultValue 请求参数的默认值
	 */
	@RequestMapping(value="/testRequestParam")
	public String testRequestParam(@RequestParam(value="username") String un,
			@RequestParam(value="age", required=false, defaultValue="0") int age) {
		System.out.println("testRequestParam, username: " + un + ", age: " + age);
		return SUCCESS;
	}
	
	/**
	 * 了解：
	 * 映射请求头信息
	 * 用法同@RequestParam
	 */
	@RequestMapping("/testRequestHeader")
	public String testRequestHeader(@RequestHeader(value="Accept-Language") String al) {
		System.out.println("testRequestHeader, Accept-Language: " + al);
		return SUCCESS;
	}
	
	/**
	 * 了解：
	 * @CookieValue:映射一个Cookie值，属性同@RequestParam
	 */
	@RequestMapping("/testCookieValue")
	public String testCookieValue(@CookieValue("JSESSIONID") String sessionId) {
		System.out.println("testCookieValue: sessionId: " + sessionId);
		return SUCCESS;
	}
	
```
<br>

### 使用 POJO 对象绑定请求参数值
```
	<form action="springmvc/testPojo" method="POST">
		username:<input type="text" name="username"/>
		<br>
		password:<input type="password" name="password"/>
		<br>
		email:<input type="text" name="email"/>
		<br>
		age:<input type="text" name="age"/>
		<br>
		city:<input type="text" name="address.city"/>
		<br>
		province:<input type="text" name="address.province"/>
		<br>
		<input type="submit" value="Submit"/>
	</form>
	
	
	/**
	 * SpringMVC会按请求参数名和POJO属性名进行自动匹配，自动为该对象填充属性值，支持级联属性
	 */
	@RequestMapping("/testPojo")
	public String testPojo(User user) {
		System.out.println("testPojo: " + user);
		return SUCCESS;
	}
```
<br>

### 使用 Servlet API 作为入参
```

<a href="springmvc/testServletAPI">Test ServletAPI</a>


	/**
	 * 可以使用Servlet 原生的API作为目标方法的参数
	 * 具体支持以下类型
	 * HttpServletRequest
	 * HttpServletResponse
	 * HttpSession
	 * java.security.Principal
	 * Locale InputStream
	 * OutputStream
	 * Reader
	 * Writer
	 * @throws IOException 
	 */
	@RequestMapping("/testServletAPI")
	public void testServletAPI(HttpServletRequest request,
			HttpServletResponse response, Writer out) throws IOException {
		System.out.println("testServletAPI, " + request + ",||| " + response);
		out.write("hello springmvc");
		//return SUCCESS;
	}
```
<br>

## 处理模型数据
Spring MVC 提供了以下几种途径输出模型数据：
* ModelAndView:
* Map 和 Model
* @SessionAttributes
* @ModelAttribute
```
	/**
	 * 目标方法的返回值可以是ModelAndView类型
	 * 其中可以包含视图和模型信息
	 * SpringMVC 会把ModelAndView的 model中数据放入到request 域对象中。
	 */
	@RequestMapping("/testModelAndView")
	public ModelAndView testModelAndView() {
		String viewName = SUCCESS;
		ModelAndView modelAndView = new ModelAndView(viewName);
		
		//添加模型数据到ModelAndView中
		modelAndView.addObject("time", new Date());
		return modelAndView;
	}
	
	
	/**
	 * 目标方法可以添加Map 类型(实际上也可以是Model类型或 ModelMap类型)的参数
	 */
	@RequestMapping("/testMap")
	public String testMap(Map<String, Object> map) {
		System.out.println(map.getClass().getName());
		map.put("names", Arrays.asList("Tom","Jerry","Mike"));
		return SUCCESS;
	}
	
	/**
	 * @SessionAttributes 除了可以通过属性名指定需要放到会话中的属性外(实际上使用的是value属性值),
	 * 还可以通过模型属性的对象类型指定哪些模板属性需要放到会话中(实际上使用的是types属性值)
	 * map.put(String, Object)
	 *
	 * @SessionAttributes(value={"user"}, types= {String.class})
	 * @RequestMapping("/springmvc")
	 * @Controller
	 * public class SpringMVCTest { XXXX
	 * 注意：该注解只能放在类的上面，而不能放在方法的上面。
	 * types属性看的是map中key的类型
	 */
	@RequestMapping("/testSessionAttributes")
	public String testSessionAttributes(Map<String,Object> map) {
		User user = new User(1,"Tom", "123456", "tom@jiu.com", 15);
		map.put("user", user);
		map.put("school", "atguigu");
		return SUCCESS;
	}
	
	
```
<br>

@ModelAttribute需要单独拉出来讲一讲:<br>
![无法加载图片](https://github.com/Ywfy/Learning-summary-for-SpringMVC/blob/master/RequestMapping/1.PNG)<br>
![无法加载图片](https://github.com/Ywfy/Learning-summary-for-SpringMVC/blob/master/RequestMapping/2.PNG)<br>
<br>
我们希望我们没有传入值的属性能够保持从数据库中取出来的值，此时就需要使用@ModelAttribute
```
/**
	 * 1、有@ModelAttribute标记的方法，会在每个目标方法执行之前被SpringMVC调用!
	 * 2、@ModelAttribute注解也可以来修饰目标方法POJO类型的入参，其value属性值有如下的作用:
	 * 	1).SpringMVC会使用value属性值在implicitModel中查找对应的对象，若存在则会直接传入到目标方法的入参中。
	 *  2).SpringMVC会以value为key，POJO类型的对象为value，存入到request中
	 */
	@ModelAttribute
	public void getUser(@RequestParam(value="id", required=false) Integer id,
			Map<String,Object> map) {
		System.out.println("modelAttribute method");
		if(id!= null) {
			//模拟从数据库中获取对象
			User user = new User(1, "Tom", "123456", "tom@atguigu.com", 12);
			System.out.println("从数据库中获取一个对象: " + user);
			
			map.put("user", user);
		}
	}
	
	
	/**
	 * 了解：
	 * 运行流程:
	 * 1、执行@ModelAttribute注解修饰的方法：从数据库中取出对象，把对象放入到了Map中，键为:user
	 * 2、SpringMVC 从Map中取出User对象，并把表单的请求参数赋给该User对象的对应属性
	 * 3、SpringMVC 把上述对象传入目标方法的参数
	 * *注意:在@ModelAttribute修改的方法中，放入到Map时的键需要和目标方法入参类型的第一个字母小写的字符串一致。
	 * 
	 * SpringMVC 确定目标方法POJO类型入参的过程
	 * 1、确定一个key
	 * 	1)若目标方法的POJO类型的参数没有使用@ModelAttribute作为修饰，则key为POJO类名第一个字母小写的字符串
	 *  2)若使用了@ModelAttribute来修饰，则key为@ModelAttribute注解的value属性值。
	 * 2、在implicitModel中查找key对应的对象，若存在，则作为入参传入
	 *  1)若在@ModelAttribute标记的方法中在Map中保存过，且key和确定的key一致，则会获取到。
	 * 3、若implicitModel中不存在key对应的对象，则检查当前的Handler是否使用@SessionAttributes注解修饰，
	 * 若使用了该注解，且@SessionAttributes注解的value属性值中包含了key，则会从HttpSession中来获取key所
	 * 对应的value值，若存在则直接传入到目标方法的入参中，若不存在则抛出异常。
	 * 4、若Handler没有标识@SessionAttributes注解或@SessioniAttributes注解的value值中不包含key，则
	 * 会通过反射来创建POJO类型的参数，传入为目标方法的参数
	 * 5、SpringMVC会把key和POJO类型的对象保存到implicitModel中，进而会保存到request中。
	 * 
	 * 源代码分析的流程:
	 * 1、调用@ModelAttribute注解修饰的方法，实际上把@ModelAttribute方法中Map中的数据放在了implicitModel中。
	 * 2、解析请求处理器的目标参数 ，实际上该目标参数来自于WebDataBinder对象的target属性
	 * 	1).创建WebDataBinder对象:
	 * 		①确定objectName属性：若传入的attrName属性值为""，则objectName为类名第一个字母小写
	 * 			注意：attrName，若目标方法的POJO属性使用了@ModelAttribute来修饰，则attrName值即为@ModelAttribute
	 * 				的value属性值	
	 * 		②确定target属性:
	 * 		>在implicitModel中查找attrName对应的属性值，若存在，OK
	 * 		>*若不存在，则验证当前Handler是否使用了@SessionAttributes进行修饰，若使用了，则尝试从Session
	 * 				中获取attrName所对应的属性值，若session中没有对应的属性值，则抛出异常
	 * 		>若Handler没有使用@SessionAttributes进行修饰，或@SessionAttributes中没有使用value值指定的key
	 * 				和attrName相匹配，则通过反射创建POJO
	 *  2).SpringMVC把表单的请求参数赋给了WebDataBinder的target对应的属性
	 *  3).*SpringMVC会把WebDataBinder的attrName和target给到implicitModel,进而传到request域对象中。
	 *  4).把WebDataBinder的target作为参数传递给目标方法的入参。
	 * 
	 */
	@RequestMapping("/testModelAttribute")
	public String testModelAttribute(User user) {
		System.out.println("修改: " + user);
		return SUCCESS;
	}
```
<br>

## 视图解析器
InternalResourceViewResolver:<br>
```
	<!-- 配置视图解析器:如何把handler方法返回值解析为实际的物理视图 -->
	<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<property name="prefix" value="/WEB-INF/views/"></property>
		<property name="suffix" value=".jsp"></property>
	</bean>
```
<br>


## 使网页语言国际化
```
<!-- 配置国际化资源文件 -->
	<bean id="messageSource"
		class="org.springframework.context.support.ResourceBundleMessageSource">
		<property name="basename" value="i18n"></property>
	</bean>

在需要的页面上声明
<%@ taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
```
此时，需要在类路径下创建国际化文件
```
i18n_en_US.properties
i18n_zh_CN.properties
i18n.properties
```
以下是i18n_en_US.properties的内容:
```
i18n.username=Username
i18n.password=Password
```
完成以后步骤后，以后我们使用i18n里声明的变量时，其值就会根据浏览器语言的不同而不同。<br>
<br>

## <mvc:view-controller>
若希望不经过处理器，而直接访问静态资源
```
<mvc:view-controller path="/success" view-name="success"/>
```
通过以上方式就可以,但是会出现只能访问success视图，而无法访问其他视图,此时加入以下标签即可
```
<mvc:annotation-driven></mvc:annotation-driven>
```
