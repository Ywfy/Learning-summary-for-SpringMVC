# 文件上传
## 步骤1：引入jar包
```
commons-fileupload-1.2.1.jar
commons-io-2.0.jar
```
## 步骤2：配置MultipartResolver
```
  <bean id="multipartResolver"
		class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
		<!-- 必须和用户 JSP 的 pageEncoding 属性一致，以便正确解析表单的内容 -->
		<property name="defaultEncoding" value="UTF-8"></property>
		<!-- 以KB为单位 -->
		<property name="maxUploadSize" value="1024000"></property>
	</bean>
```
## 步骤3:写表单
修改index.jsp
```
 <form action="testFileUpload" method="POST" enctype="multipart/form-data">
		File:<input type="file" name="file" />
		Desc:<input type="text" name="desc" />
		<input type="submit" value="submit" />
	</form>
```

## 步骤4:编写处理器方法
```
@RequestMapping("/testFileUpload")
	public String testFileUpload(@RequestParam("desc") String desc,
			@RequestParam("file") MultipartFile file) throws IOException {
		System.out.println("desc: " + desc);
		System.out.println("OriginalFilename: " + file.getOriginalFilename());
		System.out.println("InputStream: " + file.getInputStream());
		return SUCCESS;
	}
```
MultipartFile对象有以下方法<br>
![图片无法加载](https://github.com/Ywfy/Learning-summary-for-SpringMVC/blob/master/FileUpload/mult.PNG)<br>

## 多文件上传
```
@RequestMapping("/testFileUpload")
	public String testFileUpload(@RequestParam("desc") String desc,
			@RequestParam("file") MultipartFile[] file) throws IOException {
		xxx
	}

```
多文件上传非常简单，只需将MultipartFile变为MultipartFile[]就行了
若想要将文件保存在服务器，调用transferTo(File)方法就行。
