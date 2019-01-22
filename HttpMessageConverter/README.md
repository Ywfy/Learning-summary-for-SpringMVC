# HttpMessageConverter
## HttpMessageConverter原理
![图片无法加载](https://github.com/Ywfy/Learning-summary-for-SpringMVC/blob/master/Employee/img/stm.png)<br>

## HttpMessageConverter使用
* 使用 @RequestBody / @ResponseBody 对处理方法进行标注
* 使用 HttpEntity<T> / ResponseEntity<T> 作为处理方法的入参或返回值<br>
Spring 首先根据请求头或响应头的Accept 属性选择匹配的 HttpMessageConverter, 进而根据参数类型或泛型类型的过滤得到匹配的 HttpMessageConverter<br>

### @RequestBody / @ResponseBody简单示例
修改index.jsp
```
添加以下代码

	<form action="testHttpMessageConverter" method="POST" enctype="multipart/form-data">
		File:<input type="file" name="file" />
		Desc:<input type="text" name="desc" />
		<input type="submit" value="submit" />
	</form>
```
编写处理器代码
```
	@ResponseBody
	@RequestMapping("/testHttpMessageConverter")
	public String testHttpMessageConverter(@RequestBody String body) {
		System.out.println(body);
		return "helloworld! " + new Date();
	}
```
运行<br>
略

### ResponseEntity<T>简单示例
编写请求,修改index.jsp
```
<a href="testResponseEntity">test ResponseEntity</a>
```

编写处理器
```
@RequestMapping("/testResponseEntity")
	public ResponseEntity<byte[]> testResponseEntity(HttpSession session) throws IOException{
		byte[] body = null;
		ServletContext servletContext = session.getServletContext();
		InputStream in = servletContext.getResourceAsStream("/files/English.txt");
		body = new byte[in.available()];
		in.read(body);
		
		HttpHeaders headers = new HttpHeaders();
		headers.add("Content-Disposition", "attachment;filename=English.txt");
		
		HttpStatus statusCode = HttpStatus.OK;
		
		ResponseEntity<byte[]> response = new ResponseEntity<byte[]>(body, headers, statusCode);
		return response;
	}
```

运行<br>
点击test ResponseEntity，后出现<br>
![无法加载图片](https://github.com/Ywfy/Learning-summary-for-SpringMVC/blob/master/Employee/img/xz.png)<br>
注意：我在WebContent目录下新建了一个files目录，里面放了个English.txt<br>
<br>
