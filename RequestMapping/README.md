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

## REST风格请求方式

