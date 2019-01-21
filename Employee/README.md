我们通过一个员工信息增添改查系统来解说SpringMVC知识点

# 显示所有员工信息
#### 1、创建Dynamic Web Project,编写web.xml并创建springDispatcherServlet，引入jar包，创建SpringMVC配置文件
## 2、创建entity
```
Employee.java:
package com.atguigu.springmvc.crud.entities;

import java.util.Date;

import javax.validation.constraints.Past;

import org.hibernate.validator.constraints.Email;
import org.hibernate.validator.constraints.NotEmpty;
import org.springframework.format.annotation.DateTimeFormat;
import org.springframework.format.annotation.NumberFormat;

public class Employee {

	private Integer id;
	@NotEmpty
	private String lastName;

	@Email
	private String email;
	//1 male, 0 female
	private Integer gender;
	
	private Department department;
	
	@Past
	@DateTimeFormat(pattern="yyyy-MM-dd")
	private Date birth;
	
	@NumberFormat(pattern="#,###,###.#")
	private Float salary;

	public Integer getId() {
		return id;
	}

	public void setId(Integer id) {
		this.id = id;
	}

	public String getLastName() {
		return lastName;
	}

	public void setLastName(String lastName) {
		this.lastName = lastName;
	}

	public String getEmail() {
		return email;
	}

	public void setEmail(String email) {
		this.email = email;
	}

	public Integer getGender() {
		return gender;
	}

	public void setGender(Integer gender) {
		this.gender = gender;
	}

	public Department getDepartment() {
		return department;
	}

	public void setDepartment(Department department) {
		this.department = department;
	}

	public Date getBirth() {
		return birth;
	}

	public void setBirth(Date birth) {
		this.birth = birth;
	}

	public Float getSalary() {
		return salary;
	}

	public void setSalary(Float salary) {
		this.salary = salary;
	}

	@Override
	public String toString() {
		return "Employee [id=" + id + ", lastName=" + lastName + ", email="
				+ email + ", gender=" + gender + ", department=" + department
				+ ", birth=" + birth + ", salary=" + salary + "]";
	}

	public Employee(Integer id, String lastName, String email, Integer gender,
			Department department) {
		super();
		this.id = id;
		this.lastName = lastName;
		this.email = email;
		this.gender = gender;
		this.department = department;
	}

	public Employee() {
		// TODO Auto-generated constructor stub
	}
}
```
Department.java:
```
package com.atguigu.springmvc.crud.entities;

public class Department {

	private Integer id;
	private String departmentName;

	public Department() {
		// TODO Auto-generated constructor stub
	}
	
	public Department(int i, String string) {
		this.id = i;
		this.departmentName = string;
	}

	public Integer getId() {
		return id;
	}

	public void setId(Integer id) {
		this.id = id;
	}

	public String getDepartmentName() {
		return departmentName;
	}

	public void setDepartmentName(String departmentName) {
		this.departmentName = departmentName;
	}

	@Override
	public String toString() {
		return "Department [id=" + id + ", departmentName=" + departmentName
				+ "]";
	}
	
}
```

## 3、创建Dao，这里用map模拟数据库
EmployeeDao.java:
```
package com.atguigu.springmvc.crud.dao;

import java.util.Collection;
import java.util.HashMap;
import java.util.Map;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Repository;

import com.atguigu.springmvc.crud.entities.Department;
import com.atguigu.springmvc.crud.entities.Employee;

@Repository
public class EmployeeDao {

	private static Map<Integer, Employee> employees = null;
	
	@Autowired
	private DepartmentDao departmentDao;
	
	static{
		employees = new HashMap<Integer, Employee>();

		employees.put(1001, new Employee(1001, "E-AA", "aa@163.com", 1, new Department(101, "D-AA")));
		employees.put(1002, new Employee(1002, "E-BB", "bb@163.com", 1, new Department(102, "D-BB")));
		employees.put(1003, new Employee(1003, "E-CC", "cc@163.com", 0, new Department(103, "D-CC")));
		employees.put(1004, new Employee(1004, "E-DD", "dd@163.com", 0, new Department(104, "D-DD")));
		employees.put(1005, new Employee(1005, "E-EE", "ee@163.com", 1, new Department(105, "D-EE")));
	}
	
	private static Integer initId = 1006;
	
	public void save(Employee employee){
		System.out.println(employee.getId());
		if(employee.getId() == null){
			employee.setId(initId++);
		}
		
		employee.setDepartment(departmentDao.getDepartment(employee.getDepartment().getId()));
		employees.put(employee.getId(), employee);
	}
	
	public Collection<Employee> getAll(){
		return employees.values();
	}
	
	public Employee get(Integer id){
		return employees.get(id);
	}
	
	public void delete(Integer id){
		employees.remove(id);
	}
}

```
DepartmentDao.java:
```
package com.atguigu.springmvc.crud.dao;

import java.util.Collection;
import java.util.HashMap;
import java.util.Map;

import org.springframework.stereotype.Repository;

import com.atguigu.springmvc.crud.entities.Department;

@Repository
public class DepartmentDao {

	private static Map<Integer, Department> departments = null;
	
	static{
		departments = new HashMap<Integer, Department>();
		
		departments.put(101, new Department(101, "D-AA"));
		departments.put(102, new Department(102, "D-BB"));
		departments.put(103, new Department(103, "D-CC"));
		departments.put(104, new Department(104, "D-DD"));
		departments.put(105, new Department(105, "D-EE"));
	}
	
	public Collection<Department> getDepartments(){
		return departments.values();
	}
	
	public Department getDepartment(Integer id){
		return departments.get(id);
	}
	
}
```

## 4、编写Controller
```
package com.atguigu.springmvc.crud.handlers;

import java.util.Map;

import javax.validation.Valid;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.validation.BindingResult;
import org.springframework.validation.FieldError;
import org.springframework.web.bind.WebDataBinder;
import org.springframework.web.bind.annotation.InitBinder;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;

import com.atguigu.springmvc.crud.dao.DepartmentDao;
import com.atguigu.springmvc.crud.dao.EmployeeDao;
import com.atguigu.springmvc.crud.entities.Employee;

@Controller
public class EmployeeHandler {
	
	@Autowired
	private EmployeeDao employeeDao;
	
	@Autowired
	private DepartmentDao departmentDao;
	
	@RequestMapping("/emps")
	public String list(Map<String, Object> map) {
		map.put("employees", employeeDao.getAll());
		return "list";
	}

}
```
## 5、编写SpringMVC.xml
```
  <!-- 配置自动扫描的包 -->
	<context:component-scan base-package="com.atguigu.springmvc"></context:component-scan>

	<!-- 配置视图解析器 -->
	<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<property name="prefix" value="/WEB-INF/views/"></property>
		<property name="suffix" value=".jsp"></property>
	</bean>
```

## 6、编写index.jsp 和 list.jsp
index.jsp:
```
<body>
  <a href="emps">List All Employees</a>
</body>
```
list.jsp:
```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
	
	 <form action="" method="POST"></form> 
	
	<c:if test="${empty requestScope.employees}">
		没有任何员工信息
	</c:if>
	<c:if test="${!empty requestScope.employees}">
		<table border="1" cellpadding="10" cellspacing="0">
			<tr>
				<th>ID</th>
				<th>LastName</th>
				<th>Email</th>
				<th>Gender</th>
				<th>Department</th>
				<th>Edit</th>
				<th>Delete</th>
			</tr>
			<c:forEach items="${requestScope.employees}" var="emp">
				<tr>
					<td>${emp.id }</td>
					<td>${emp.lastName }</td>
					<td>${emp.email }</td>
					<td>${emp.gender == 0 ? 'Female' : 'Male' }</td>
					<td>${emp.department.departmentName }</td>
					<td><a href="emp/edit/${emp.id}">Edit</a></td>
					<td><a class="delete" href="emp/delete/${emp.id }">Delete</a></td>
				</tr>
			</c:forEach>
		</table>
	</c:if>
	
</body>
</html>
```

## 运行
点击index.jsp的List All Employees的超链接后，跳转到以下页面<br>
![无法加载图片](https://github.com/Ywfy/Learning-summary-for-SpringMVC/blob/master/Employee/all.png)<br>
<br>

# 添加员工功能
## 1、在list.jsp上添加超链接
```
<a href="emp">Add New Employee</a>
```

## 2、在EmployeeHandler下编写
```
  @RequestMapping(value="/emp", method=RequestMethod.GET)
	public String input(Map<String, Object> map) {
		map.put("departments", departmentDao.getDepartments());
		map.put("employee", new Employee());
		return "input";
	}
  之所以要多这一步，是为了下面的输入页面做准备
```

## 3、编写添加页面input.jsp
```
<%@page import="java.util.Map"%>
<%@page import="java.util.HashMap" %>
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>

<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
<!-- 
	1、为什么需要使用form标签 
		可以更快速地开发出表单页面，而且可以更方便的进行表单值的回显
	2、注意：可以通过modelAttribute属性指定绑定的模型属性，若没有指定该属性，
		   则默认从request域对象中读取command的表单bean，如果该属性值也不存在，则会发生错误。
		 简单点说，就是request域中必须有一个和表单字段对应的一个bean
		   处理方法：form标签加modelAttribute="employee"
		           处理器中map.put("employee", new Employee());
	-->
	<form:form action="${pageContext.request.contextPath }/emp" method="POST" modelAttribute="employee">

		<!-- path属性对应于html表单标签的name属性值 -->
    <!-- Spring的form标签强制表单回显，所以form里面的path属性必须写modelAttribute所写对象拥有的属性 -->
		LastName:<form:input path="lastName"/>
		<form:errors path="lastName"></form:errors>
	
		<br>
		Email:<form:input path="email"/>
		<form:errors path="email"></form:errors>
		<br>
    
		<%
			Map<String, String> genders = new HashMap();
			genders.put("1", "Male");
			genders.put("0", "Female");
			
			request.setAttribute("genders", genders);
		%>
		Gender:
		<br>
		<form:radiobuttons path="gender" items="${genders }" delimiter="<br>"/>
		<br>
    
		department:<form:select path="department.id" items="${departments }"
			itemLabel="departmentName" itemValue="id"></form:select>
		<br>
    <input type="submit" value="submit"/>
  </form:form>
</body>
</html>
```

## 4、在EmployeeHandler编写emp,但method为POST的方法用来保存对象
```
  @RequestMapping(value= {"/emp"}, method=RequestMethod.POST)
	public String save(Employee employee) {
		System.out.println("save: " + employee);
		employeeDao.save(employee);
		return "redirect:/emps";
	}
```

## 5、运行
点击list.jsp页面的Add New Employee超链接，来到添加页面<br>
![图片无法加载](https://github.com/Ywfy/Learning-summary-for-SpringMVC/blob/master/Employee/%E8%BE%93%E5%85%A51.png)<br>
输入信息后，点击Submit<br>
![图片无法加载](https://github.com/Ywfy/Learning-summary-for-SpringMVC/blob/master/Employee/img/all2.png)<br>
<br>

# 实现修改功能
之前的Edit超链接只是摆设，现在我们来实现它<br>
## 1、修改list.jsp
```
<td><a href="emp/${emp.id}">Edit</a></td>
```

## 2、在EmployeeHandler编写处理方法
```
  @RequestMapping(value="/emp/{id}", method=RequestMethod.GET)
	public String input(@PathVariable("id") Integer id, Map<String, Object> map) {
		map.put("employee", employeeDao.get(id));
		System.out.println(map.get("employee"));
		map.put("departments", departmentDao.getDepartments());
		return "input";
	}
```

## 3、修改input.jsp
此处我们提出一个需求：LastName是不可修改的，对于修改页面，LastName输入项应该隐藏<br>
对于这个需求，使用<c:if>标签就可以搞定,通过判定传过来的Employee的id是否为空
```
<c:if test="${employee.id == null }">
			<!-- path属性对应于html表单标签的name属性值 -->
			LastName:<form:input path="lastName"/>
</c:if>
<c:if test="${employee.id != null }">
			<form:hidden path="id"/>
			<!-- Spring的form标签强制表单回显，所以form里面的path属性必须写modelAttribute所写对象拥有的属性 -->
</c:if>
<br>
```

## 运行
点击Edit，来到修改页面<br>
![无法加载图片](https://github.com/Ywfy/Learning-summary-for-SpringMVC/blob/master/Employee/img/xg1.png)<br>
![无法加载图片](https://github.com/Ywfy/Learning-summary-for-SpringMVC/blob/master/Employee/img/xgb.png)<br>
修改信息后，点击Submit<br>
![无法加载图片](https://github.com/Ywfy/Learning-summary-for-SpringMVC/blob/master/Employee/img/xga.png)<br>
<br>

# 添加删除功能
之前的delete超链接同样只是一个摆设，我们来实现它<br>
## 1、修改list.jsp
这里老司机打算开一波车，我们将delete的超链接同样设为"emp/{id}",这时肯定会和Edit相冲突，怎么办呢?
```
<td><a class="delete" href="emp/${emp.id }">Delete</a></td>
```
这里我们利用JS代码来实现GET到POST的转变，就不会冲突了
```
<script type="text/javascript" src="scripts/jquery-1.9.1.min.js"></script>
<script type="text/javascript">
	$(function(){
		$(".delete").click(function(){
			var href = $(this).attr("href");
			$("form").attr("action", href).submit();
			return false;
		})
	})
</script>
```
此处需要在WebContent下新建scripts文件夹，将jquery-1.9.1.min.js放进去,下载链接在此===>[jquery-1.9.1.min.js](https://github.com/Ywfy/Learning-summary-for-SpringMVC/blob/master/Employee/jquery-1.9.1.min.js)<br>

## 2、在EmployeeHandler下创建处理方法
```
@RequestMapping(value="/emp/{id}", method=RequestMethod.POST)
	public String delete(@PathVariable("id") Integer id) {
		employeeDao.delete(id);
		return "redirect:/emps";
	}
```

## 运行
点击delete<br>
![图片无法加载](https://github.com/Ywfy/Learning-summary-for-SpringMVC/blob/master/Employee/all.png)<br>
<br>

# 给员工添加出生日期属性 和 薪水 属性
想要实现这个功能，出现了一个大问题，就是我们的输入肯定是字符串属性的，然而出生日期是 Date。这里涉及到 字符串到Date的转化问题<br>
而且字符串到Date的转化肯定要涉及到字符串的格式问题，什么格式的String能满足要求？<br>
若我们输入的String格式不合格怎么办？要有数据检验<br>
其实前面的LastName、Email等也都需要数据校验，不然LastName给个$ ,Email给个1 ，那肯定是不合理的。<br>
<br>

<strong>数据绑定流程</strong>:
![图片无法加载](https://github.com/Ywfy/Learning-summary-for-SpringMVC/blob/master/Employee/img/%E6%95%B0%E6%8D%AE%E7%BB%91%E5%AE%9A%E6%B5%81%E7%A8%8B.png)<br>

## 1、类型转换器
SpringMVC已经内置了很多类型转换器，可完成大多数Java类型的转换工作：<br>
只有涉及到我们自己创建的类时才考虑使用自定义类型转换器。<br>
这里我们提出一个需求，给一个输入框，输入lastname-email-gender-depaerment.id 例如：GG-gg@aiguigu.com-0-105，来创建并保存Employee对象<br>

在input.jsp添加:
```
	<form action="emp/testConversionServiceConverer" method="POST"> 
		<!-- lastname-email-gender-depaerment.id 例如：GG-gg@aiguigu.com-0-105-->
		Employee:<input type="text" name="employee"/>
		<input type="submit" value="submit"/>
	</form>
```
创建对应的处理方法:
```
@RequestMapping(value="emp/testConversionServiceConverer", method=RequestMethod.POST)
public String testConverter(@RequestParam("employee") Employee employee) {
	System.out.println("save: " + employee);
	employeeDao.save(employee);
	return "redirect:/emps";
}
```
@RequestParam("employee")取出的是一个String，然而我们的参数却是Employee，这里就需要一个自定义类型转换器来转换<br>


### 自定义类型转换器
#### 1)、实现Converter<S, T>接口
```
package com.atguigu.springmvc.converters;

import org.springframework.core.convert.converter.Converter;
import org.springframework.stereotype.Component;

import com.atguigu.springmvc.crud.entities.Department;
import com.atguigu.springmvc.crud.entities.Employee;

@Component
public class EmployeeConverter implements Converter<String, Employee> {

	@Override
	public Employee convert(String source) {
		// TODO Auto-generated method stub
		if(source != null) {
			//GG-gg@aiguigu.com-0-105
			String[] vals = source.split("-");
			if(vals != null && vals.length == 4) {
				String lastName = vals[0];
				String email = vals[1];
				Integer gender = Integer.parseInt(vals[2]);
				Department department = new Department();
				department.setId(Integer.parseInt(vals[3]));
				
				Employee employee = new Employee(null, lastName, email, gender, department);
				System.out.println(source + " convert " + employee);
				return employee;
			}
		}
		return null;
	}	

}
```

#### 2)、在SpringMVC配置文件配置ConversionService
```
 	<!-- 配置时一定要加上 -->
	<mvc:annotation-driven conversion-service="conversionService"></mvc:annotation-driven>
	
	<!-- 配置ConversionService -->
	<!-- org.springframework.format.support.FormattingConversionServiceFactoryBean
		可以使我们既可以使用自定义转换器，也可以使用spring自带的格式化服务	
	 -->
	<bean id="conversionService"
		class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
		<property name="converters">
			<set>
				<ref bean="employeeConverter"/>
			</set>
		</property>
	</bean>
```

## 运行
![无法加载图片](https://github.com/Ywfy/Learning-summary-for-SpringMVC/blob/master/Employee/img/za.png)<br>
![无法加载图片](https://github.com/Ywfy/Learning-summary-for-SpringMVC/blob/master/Employee/img/zar.png)<br>

## 扩展：@InitBinder
@InitBinder用于在@Controller中标注于方法上，表示为当前控制器注册一个属性编辑器，只对当前的Controller有效。<br>
@InitBinder标注的方法必须有一个参数WebDataBinder。所谓的属性编辑器可以理解就是帮助我们完成参数绑定。<br>
@InitBinder很有用，我们这里先不细讲，用一个例子简单演示一下<br>
在EmployeeHanler下添加：
```
	@InitBinder
	public void initBinder(WebDataBinder binder) {
		binder.setDisallowedFields("lastName");
	}
```
运行效果:<br>
![无法加载图片](https://github.com/Ywfy/Learning-summary-for-SpringMVC/blob/master/Employee/img/b.png)<br>
这里我是有输入LastName的，但是最终却并没有被绑定保存。<br>
<br>


## 字符串格式化
其实非常简单，两步搞定<br>
### 第一步：全局配置\<mvc:annotation-driven></mvc:annotation-driven>
这里前面已经配了，就不用了
### 第二步：在Bean需要的属性上加上注解
修改Employee：
```
	@DateTimeFormat(pattern="yyyy-MM-dd")
	private Date birth;
	
	@NumberFormat(pattern="#,###,###.#")
	private Float salary;
```
## 运行
输入<br>
日期为2015-12-12<br>
薪水为1,234,567.8<br>
```
save: Employee [id=null, lastName=null, email=gg@789.com, gender=0, department=Department [id=102, departmentName=null], birth=Sat Dec 12 00:00:00 CST 2015, salary=1234567.8]
```
<br>

## 数据校验
* 实际上Java 为 Bean 数据合法性校验提供了标准框架JSR303
* JSR 303 通过在 Bean 属性上标注类似于 @NotNull、@Max 等标准的注解指定校验规则，并通过标准的验证接口对 Bean 进行验证<br>
![图片无法加载](https://github.com/Ywfy/Learning-summary-for-SpringMVC/blob/master/Employee/img/jsr303.jpg)<br><br>
* JSR303只是一个标准，Hibernate Validator 是 JSR 303 的一个参考实现,除支持所有标准的校验注解外，它还支持以下的扩展注解<br>
![图片无法加载](https://github.com/Ywfy/Learning-summary-for-SpringMVC/blob/master/Employee/img/ha.png)<br><br>
* <strong>使用流程:</strong>
### 1)、引入jar包
```
validation-api-1.1.0.CR1.jar
hibernate-validator-5.0.0.CR2.jar
hibernate-validator-annotation-processor-5.0.0.CR2.jar
```

### 2)、在SpringMVC配置文件中添加\<mvc:annotation-driven></mvc:annotation-driven>
### 3)、需要在bean的属性上添加对应的注解
```
public class Employee {

	private Integer id;
	@NotEmpty
	private String lastName;

	@Email
	private String email;
	//1 male, 0 female
	private Integer gender;
	
	private Department department;
	
	@Past
	@DateTimeFormat(pattern="yyyy-MM-dd")
	private Date birth;
	
	@NumberFormat(pattern="#,###,###.#")
	private Float salary;

	......
```

### 4)、在目标方法bean类型的前面添加@Valid注解,并在随后添加一个BindingResult或者error对象
在处理方法对应的入参前添加 @Valid，Spring MVC 就会实施校验并将校验结果保存在被校验入参对象之后的 BindingResult 或Errors 入参中<br>
注意:需校验的Bean对象和其绑定结果对象(BindingResult)或错误对象(error)是成对出现的，它们中间不允许声明其他的入参<br>

BindingResult类型对象有以下常用方法：
* FieldError getFieldError(String field)
* List<FieldError> getFieldErrors()
* Object getFieldValue(String field)
* Int getErrorCount()
<br>
	
修改EmployeeHandler:
```
	@RequestMapping(value= {"/emp"}, method=RequestMethod.POST)
	public String save(@Valid Employee employee, BindingResult result,
			Map<String, Object> map) {
		System.out.println("save: " + employee);
		
		if(result.getErrorCount() > 0) {
			System.out.println("出错了");
			for(FieldError error:result.getFieldErrors()) {
				System.out.println(error.getField() + ":" + error.getDefaultMessage());
			}
			
			//若验证出错，则转向定制的页面
			map.put("departments", departmentDao.getDepartments());
			return "input";
		}
		employeeDao.save(employee);
		return "redirect:/emps";
	}
```
以上虽然已经完成了验证，但是验证出错信息却是打印在控制台，我们希望将错误信息实时地显示在页面上
### 5)、错误信息实时打印在页面上
Spring MVC 除了会将表单/命令对象的校验结果保存到对应的 BindingResult 或 Errors 对象中外，还会将所有校验结果保存到 “隐含模型”<br>
在 JSP 页面上可通过 <form:errors path=“userName”> 自动获取错误信息并显示<br>

修改input.jsp:
```
	<form:form action="${pageContext.request.contextPath }/emp" method="POST" modelAttribute="employee">
		
		<c:if test="${employee.id == null }">
			<!-- path属性对应于html表单标签的name属性值 -->
			LastName:<form:input path="lastName"/>
			<form:errors path="lastName"></form:errors>
		</c:if>
		<c:if test="${employee.id != null }">
			<form:hidden path="id"/>
			<!-- Spring的form标签强制表单回显，所以form里面的path属性必须写modelAttribute所写对象拥有的属性 -->
		</c:if>
		<br>
		
		Email:<form:input path="email"/>
		<form:errors path="email"></form:errors>
		<br>
		
		<%
			Map<String, String> genders = new HashMap();
			genders.put("1", "Male");
			genders.put("0", "Female");
			
			request.setAttribute("genders", genders);
		%>
		Gender:
		<br>
		<form:radiobuttons path="gender" items="${genders }" delimiter="<br>"/>
		<br>
		
		department:<form:select path="department.id" items="${departments }"
			itemLabel="departmentName" itemValue="id"></form:select>
		<br>
		
		Birth:<form:input path="birth"/>
		<form:errors path="birth"></form:errors>
		<br>
		
		Salary:<form:input path="salary"/>
		<br>
		<input type="submit" value="submit"/>
	</form:form>
```

### 运行:
![图片无法加载](https://github.com/Ywfy/Learning-summary-for-SpringMVC/blob/master/Employee/img/er1.png)

### 6)、错误提示消息国际化
这里其实我们可以自定义错误提示消息<br>
#### 步骤1：创建i18n配置文件
i18n_zh_CN.properties:
```
NotEmpty.employee.lastName=^^LastName\u4E0D\u80FD\u4E3A\u7A7A
Email.employee.email=Email\u5730\u5740\u4E0D\u5408\u6CD5
Past.employee.birth=Birth\u4E0D\u80FD\u662F\u5C06\u6765\u7684\u65F6\u95F4

typeMismatch.employee.birth=Birth\u4E0D\u662F\u4E00\u4E2A\u65E5\u671F
```

注意上方右边的那些看着像乱码的东西，其实我们打进去的是中文，只是他自动显示成这样子<br>
<br>
错误消息在国际资源配置文件的写法:<br>
&nbsp;&nbsp;&nbsp;以键值对的形式，值为提示的错误消息<br>
&nbsp;&nbsp;&nbsp;键为检验注解作为前缀,请求域里的Bean属性名,校验的属性名<br>
&nbsp;&nbsp;&nbsp;例:NotEmpty.employee.lastName=不能为空<br>
<br>
i18n_en_US.properties：
```
NotEmpty.employee.lastName=LastName can not be Empty
Email.employee.email=Email is not valid
Past.employee.birth=Birth must be the past


typeMismatch.employee.birth=Your input is not a date
```
i18n.properties:
```
NotEmpty.employee.lastName=LastName\u4E0D\u80FD\u4E3A\u7A7A
Email.employee.email=Email\u5730\u5740\u4E0D\u5408\u6CD5
Past.employee.birth=Birth\u4E0D\u80FD\u662F\u5C06\u6765\u7684\u65F6\u95F4

typeMismatch.employee.birth=Birth\u4E0D\u662F\u4E00\u4E2A\u65E5\u671F
```
这里顺便说一下，有三个错误代码是可以不在Bean的属性上标注就可以用的：
```
required：必要的参数不存在。如 @RequiredParam(“param1”) 标注了一个入参，但是该参数不存在
typeMismatch：在数据绑定时，发生数据类型不匹配的问题
methodInvocation：Spring MVC 在调用处理方法时发生了错误
```

#### 步骤2：在springmvc配置文件 配置国际化资源文件标签
```
	<!-- 配置国际化资源文件 -->
	<bean id="messageSource"
		class="org.springframework.context.support.ResourceBundleMessageSource">
		<property name="basename" value="i18n"></property>	
	</bean>
```

#### 运行
注意：此时我将浏览器语言环境改为了英语(en_US)<br>
![图片无法加载](https://github.com/Ywfy/Learning-summary-for-SpringMVC/blob/master/Employee/img/er2.png)<br>
![图片无法加载](https://github.com/Ywfy/Learning-summary-for-SpringMVC/blob/master/Employee/img/er3.png)<br>



