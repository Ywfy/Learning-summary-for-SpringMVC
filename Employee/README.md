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
我们按照功能流程来:<br>
## 1、数据检验 & 字符串格式化
* 实际上Java 为 Bean 数据合法性校验提供了标准框架JSR303
* JSR 303 通过在 Bean 属性上标注类似于 @NotNull、@Max 等标准的注解指定校验规则，并通过标准的验证接口对 Bean 进行验证<br>
![图片无法加载](https://github.com/Ywfy/Learning-summary-for-SpringMVC/blob/master/Employee/img/jsr303.jpg)<br><br>
* JSR303只是一个标准，Hibernate Validator 是 JSR 303 的一个参考实现,除支持所有标准的校验注解外，它还支持以下的扩展注解<br>
![图片无法加载](https://github.com/Ywfy/Learning-summary-for-SpringMVC/blob/master/Employee/img/ha.png)<br><br>


