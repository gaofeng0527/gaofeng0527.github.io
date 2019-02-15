---
title: springmvc使用thymeleaf
date: 2017-11-29 12:47:05
tags: [springmvc,thymeleaf]
---

# thymeleaf简介

之前开发javaWEB项目大多都是用JSP作为页面显示，但是JSP和HTML还是有很大区别，渲染的效果有时候也是有差异的。thymeleaf和HTMl更像，去掉了JSTL和spring提供的标签，可以在web容器之外的环境下用浏览器直接打开。前端人员可以直接查看样式，当有后台传输数据过来后，会动态的替换页面中的静态内容。

<!-- more -->

# springmvc配置thymeleaf视图解析器

## 添加jar依赖

这里使用maven

```xml
<!-- https://mvnrepository.com/artifact/org.thymeleaf/thymeleaf-spring4 -->
		<dependency>
			<groupId>org.thymeleaf</groupId>
			<artifactId>thymeleaf-spring4</artifactId>
			<version>3.0.7.RELEASE</version>
		</dependency>

		<!-- https://mvnrepository.com/artifact/org.thymeleaf/thymeleaf -->
		<dependency>
			<groupId>org.thymeleaf</groupId>
			<artifactId>thymeleaf</artifactId>
			<version>3.0.9.RELEASE</version>
		</dependency>
```

## 在spring配置文件中添加视图映射

```xml
<!-- 设置thymeleaf模板解析器 -->
	<bean id="templateResolver"
		class="org.thymeleaf.spring4.templateresolver.SpringResourceTemplateResolver">
		<property name="prefix" value="/WEB-INF/templates/"></property>
		<property name="suffix" value=".html"></property>
		<property name="templateMode" value="HTML"></property>
		<property name="characterEncoding" value="UTF-8"></property>
		<property name="cacheable" value="false"></property>
	</bean>
	<!-- 设置模板引擎 -->
	<bean id="templateEngine" class="org.thymeleaf.spring4.SpringTemplateEngine">
		<property name="templateResolver" ref="templateResolver"></property>
		<property name="enableSpringELCompiler" value="true"></property>
	</bean>
	<!-- 配置thymeleaf视图解析器 -->
	<bean class="org.thymeleaf.spring4.view.ThymeleafViewResolver">
		<property name="templateEngine" ref="templateEngine"></property>
		<property name="characterEncoding" value="UTF-8" />
		<property name="order" value="1"></property>
	</bean>
```

注意：

>    1.   在开发阶段**cacheable**属性设置为false，不设置缓存，这样我们修改页面内容后，可以更快的看到变化，部署到线上环境后设置为true。
>    2.   **enableSpringELCompiler**属性用来设置是否允许前端页面使用*SpEL*
>    3.   记得设置**characterEncoding**属性，防止乱码

配置好以上内容，就可以使用thymeleaf了。编写代码测试一下

## 编写@controller

```java
package springmvc.controller;

import java.io.UnsupportedEncodingException;
import java.net.URLEncoder;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

import springmvc.pojo.User;

@Controller
@RequestMapping(value = "/person")
public class PersonController {

	public PersonController() {
		// TODO Auto-generated constructor stub
	}

	@RequestMapping(value = "/index", method = RequestMethod.GET)
	public String index() {
		return "person/index";
	}

	@RequestMapping(value = "/register", method = RequestMethod.GET)
	public String register(Model model) {
		User user = new User();
		model.addAttribute(user);
		return "person/register";
	}

	@RequestMapping(value = "register", method = RequestMethod.POST)
	public String register(User user) throws UnsupportedEncodingException {
		System.out.println(user.toString());
		return "redirect:/person/model/" + URLEncoder.encode(user.getUname(), "UTF-8")+"/"+URLEncoder.encode(user.getMome(), "utf-8");
	}

	@RequestMapping(value = "/model/{uname}/{mome}", method = RequestMethod.GET)
	public String model(@PathVariable("uname") String uname,@PathVariable("mome") String mome, Model model) {
		System.out.println(uname);
		User u = new User();
		u.setUname(uname);
		u.setMome(mome);
		model.addAttribute(u);
		return "person/model";
	}

}

```

## 编写html

index.html

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<!-- 添加 xmlns:th="http://www.thymeleaf.org"-->
<html xmlns:th="http://www.thymeleaf.org">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>welcome thymeleaf!</title>
</head>
<body>
<a th:href="@{register}">注册</a>
</body>
</html>
```

超链接使用@{}

register.html

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html xmlns:th="http://www.thymeleaf.org">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>

<form th:action="@{/person/register}" th:object="${user}" th:method="post" >
	uname:<input type="text" th:field="*{uname}" th:placeholder="请填写真实姓名"><br>
	mome:<input type="text" th:field="*{mome}"><br>
	<input type="submit" value="提 交">
</form>

</body>
</html>
```

model.html

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html xmlns:th="http://www.thymeleaf.org">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
	<form th:action="@{/person/register}" th:object="${user}"
		th:method="post">
		uname:<input type="text" th:field="*{uname}"
			th:unless="'' eq ${user.uname} or null eq ${user.uname}"
			th:placeholder="请填写真实姓名"><br> mome:<input type="text"
			th:field="*{mome}"><br> <input type="submit" value="提 交">
	</form>
</body>
</html>
```

## 运行

运行正常，图片就不帖了，初学的注意一下乱码问题，还有前端页面显示的问题。如果在controller中需要向页面中传值，前端需要有一个html表情承载才会显示例如：我需要在前端页面显示username，前端页面需要有一个lable标签

```html
<!-- 当username没有值的时候，显示张高峰，有值的时候就会显示username的值 -->
<lable th:text="${username}">张高峰</lable>
```

因为thymeleaf模板创建的页面，可作为静态资源直接在浏览器端显示，又可在运行时承载后台传输过来的数据，这种模式更好的把前端设计人员和后台开发人员工作分离，并又很好的对接。

## data-th-text

我们上面的文件感觉像是是HTML5，可以由任何浏览器正确显示，因为它不包括任何非HTML标签（浏览器忽略他们不明白的所有属性，如：th:text）。但是这个模板并不是一个真正有效的HTML5文档，因为HTML5规范不允许在th：*形式中使用这些非标准属性。 事实上，我们甚至在我们的<html>标签中添加了一个xmlns：th属性，这绝对是非HTML5标准。

那么如何使这个HTML5模板有效呢？ Easy：切换到Thymeleaf的数据属性语法，使用data-属性名称和连字符（ - ）分隔符的数据前缀而不是分号（:)：

```html
<p data-th-text="#{home.welcome}">Welcome to our grocery store!</p>
```

```html
<p th:text="#{home.welcome}">Welcome to our grocery store!</p>
```

这两个效果是一样的。