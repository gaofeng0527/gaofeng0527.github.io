---
title: Security入门
date: 2017-12-06 16:50:43
tags: [springmvc,security]
---

Spring Security是一个灵活和强大的身份验证和访问控制框架，以确保基于Spring的Java Web应用程序的安全。

<!-- more -->

# 引用jar

```xml
<dependency>
	<groupId>org.springframework.security</groupId>
	<artifactId>spring-security-web</artifactId>
	<version>4.1.3.RELEASE</version>
</dependency>
<dependency>
	<groupId>org.springframework.security</groupId>
	<artifactId>spring-security-config</artifactId>
	<version>4.1.3.RELEASE</version>
</dependency>
```

# 创建两个对比页面

hello.html

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html xmlns:th="http://www.thymeleaf.org">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title th:text="${title}">security</title>
</head>
<body>
	<h1 th:text="${message}">哈哈哈</h1>
</body>
</html>
```

admin.html

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html xmlns:th="http://www.thymeleaf.org">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title th:text="${title}">Insert title here</title>
</head>
<body>
	<h1 th:text="${message}">你好</h1>
</body>
</html>
```

# 编写controller

```java
package springmvc.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

@Controller
@RequestMapping("/home/se")
public class SecurityHelloController {

	public SecurityHelloController() {
		// TODO Auto-generated constructor stub
	}
	
	@RequestMapping(value="/hello",method=RequestMethod.GET)
	public String hello(Model model) {
		model.addAttribute("title", "spring security");
		model.addAttribute("message", "你好 Security!");
		return "hello";
	}
	
	@RequestMapping(value="/admin",method=RequestMethod.GET)
	public String admin(Model model) {
		model.addAttribute("title", "spring security");
		model.addAttribute("message", "这是一个受保护的页面!");
		return "admin";
	}

}

```

# 配置springmvc

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
	xmlns:mvc="http://www.springframework.org/schema/mvc"
	xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.2.xsd
		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.2.xsd">

	<context:component-scan base-package="springmvc.controller"></context:component-scan>
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
</beans>

```

# 编写spring-security.xml

权限定义为 `ROLE_ADMIN`，判断是否有权限使用 `hasRole('ADMIN') 或者 hasRole('ROLE_ADMIN')`，前缀 ROLE_ 可以省略
需要多个权限: `hasRole('ADMIN') and hasRole('DBA')`
有任意一个权限: `hasAnyRole('ROLE_ADMIN', 'ROLE_DBA')`

如果想使用权限列表的方式，而不是上面的这种表达式，需要设置 `<security:http auto-config="true" use-expressions="false">`，然后才能`<security:intercept-url pattern="/home/se/admin" access="ROLE_ADMIN"/>`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- 1.该配置主要用于配置Spring-Security，因此用到的security命名空间比较多，所以把security设置为默认的命名空间，而beans需要显示配置例如：<beans:beans> -->
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:security="http://www.springframework.org/schema/security"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
						http://www.springframework.org/schema/beans/spring-beans.xsd
						http://www.springframework.org/schema/security 
						http://www.springframework.org/schema/security/spring-security-4.1.xsd">

	<security:http auto-config="true" use-expressions="false">
      <!-- 1.intercept-url 可以配置请求路径需要什么样的权限访问
                1.1 pattern 配置请求路径，可以借用ant语法
                1.2 access 配置角色权限 hasRole('USER')即ROLE_USER，多个的话可以用逗号隔开
                1.3 method 可以配置对什么类型的请求进行拦截
                1.4 可以在http标签下配置多个intercept-url
             这句话的意思是：拦截所有GET请求，并验证用户身份，只允许拥有USER角色的用户通过
         -->
		<security:intercept-url pattern="/home/se/admin" access="ROLE_ADMIN"/>
	</security:http>
	<security:authentication-manager>
		<security:authentication-provider>
			<security:user-service>
              <!--
                    1.默认把用户信息配置到内存中
                    2.密码以{noop}为前缀，表示DelegatingPasswordEncode，大概意思就是用明文的方式，方便测试阅读
                -->
				<security:user name="zhanggaofeng" password="{noop}zhanggaofeng" authorities="ROLE_ADMIN"/>
			</security:user-service>
		</security:authentication-provider>
	</security:authentication-manager>
</beans>
```

# 添加Security

需要修改web.xml配置文件，添加org.springframwork.web.filter.DelegatingFilterProxy映射。配置拦截的地方主要：`<url-pattem>\*</url-pattem>`,一定是\* 而不是 \可以百度一下这两个配置的区别。

```xml
<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
<context-param>
	<param-name>contextConfigLocation</param-name>
	<param-value>classpath:spring-security.xml</param-value>
</context-param>
<filter>
	<filter-name>springSecurityFilterChain</filter-name>
	<filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>
<filter-mapping>
	<filter-name>springSecurityFilterChain</filter-name>
	<url-pattern>/*</url-pattern>
</filter-mapping>
```

# 测试

1.访问 /home/se/helo 能正常显示页面，因为Security没有限制该请求

2.访问 /home/se/admin 跳转到登录页面（我们没有创建登录页面，为什么会跳转呢，因为security默认了一个登录页面）,因为在spring-security.xml配置文件中对该请求添加了权限验证。