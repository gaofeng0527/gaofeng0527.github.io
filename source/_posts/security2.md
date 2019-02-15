---
title: Security自定义表单验证
date: 2017-12-06 17:29:53
tags: [security]
---

前面已经初步认识了Security框架，并且做了一个类似于Hello word的例子。当我们没有配置登录界面的时候，Security框架经过验证需要用户登录时，会预编译一个默认的登录界面。今天我们学习一下自定义的登录验证页面。

<!-- more -->

# Spring Security配置

在我们实际开发中，围绕登录和注销，结合Spring Security框架会有以下几种情况：

>    1.   当用户访问的资源需要登录验证，可以配置自定义的登录页面
>    2.   当用户提交登录信息，经过security的验证该用户登录信息错误，可以配置指定的URL
>    3.   处理登录请求的URL
>    4.   当用户直接访问登录页面，登录成功后跳转到什么页面
>    5.   用户注销登录页面后，跳转到什么页面
>    6.   用户登录成功，却没有权限访问该资源时，跳转到什么页面

以上情况都可以在security配置文件中配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:security="http://www.springframework.org/schema/security"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
						http://www.springframework.org/schema/beans/spring-beans.xsd
						http://www.springframework.org/schema/security 
						http://www.springframework.org/schema/security/spring-security-4.1.xsd">

	<!-- 一些不需要登录就可以访问的资源，取消验证，这样就省去了很多filter的拦截，提高程序效率 -->
	<security:http security="none" pattern="/loginPage"></security:http>
	<security:http auto-config="true" use-expressions="false">
		<security:intercept-url pattern="/admin" access="ROLE_ADMIN" />
		<!-- 配置登录验证的一些信息 
			1.login-page:get请求登录页面的url 
			2.login-processing-url：post请求，处理登录请求的URL ，这里配置的是security默认的处理方法/login（如过没有特殊处理，这里可以省略配置）
			3.default-target-url:当用户直接访问登录页面，登录成功后，跳转到该属性配置的url 
		-->
		<security:form-login 
			login-page="/loginPage"
			login-processing-url="/login" 
			default-target-url="/"
			authentication-failure-url="/login?error" 
			username-parameter="username"
			password-parameter="password" />
		<!-- 配置注销登录的一些配置 
			1.logout-url：get请求，注销的url
			2.logout-success-url:get请求，注销成功后访问的url 
		
		-->
		<security:logout 
			logout-url="/logout"
			logout-success-url="/login?logout" />
		<!-- 登录成功，却没有权限的页面 error-page:没有权限访问的url -->
		<security:access-denied-handler
			error-page="/deny" />
		<security:csrf disabled="true" />
	</security:http>
	<security:authentication-manager>
		<security:authentication-provider>
			<security:user-service>
				<security:user name="zhanggaofeng" password="zhanggaofeng"
					authorities="ROLE_ADMIN" />
				<security:user name="zhaojiong" password="zhaojiong"
					authorities="ROLE_USER" />
			</security:user-service>
		</security:authentication-provider>
	</security:authentication-manager>
</beans>
```

# 编写Controller

```java
package springmvc.controller;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.web.authentication.logout.SecurityContextLogoutHandler;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class LoginController {

	public LoginController() {
		// TODO Auto-generated constructor stub
	}

	@RequestMapping(value = "/logout", method = RequestMethod.GET)
	@ResponseBody
	public String logoutPage(HttpServletRequest request, HttpServletResponse response) {
		Authentication auth = SecurityContextHolder.getContext().getAuthentication();
		if (auth != null) {
			System.out.println(auth.toString());
			new SecurityContextLogoutHandler().logout(request, response, auth);
		}
		return "logout success";
	}
	
	@RequestMapping(value = "/loginPage", method = RequestMethod.GET)
	public String login() {
		return "login";
	}
	

	@RequestMapping(value = "/login")
	public String login(@RequestParam(value = "error", required = false) String error,
			@RequestParam(value = "logout", required = false) String logout, Model model) {
		if (error != null) {
			model.addAttribute("message", "登录失败");
		}
		if (logout != null) {
			model.addAttribute("message", "注销成功");
		}
		return "login";
	}

	@RequestMapping("/deny")
	public String deny(Model model) {
		model.addAttribute("errormessage", "您没有权限访问该页面");
		return "error";
	}

}
```

# 登录表单

```html
<html>
<head>
<title>Login Page</title>
</head>
<body onload='document.f.j_username.focus();'>
	<h3>Login with Username and Password</h3>
	<h4 th:if="${message} != '' "><span th:text="${message}"></span></h4>
  <!-- form的action默认配置为“/login” ，默认交给security内置的url处理 -->
	<form name='f' action='/login' method='POST'>
		<table>
			<tr>
				<td>User:</td>
				<td><input type='text' name='username' value=''></td>
			</tr>
			<tr>
				<td>Password:</td>
				<td><input type='password' name='password' /></td>
			</tr>
			<tr>
				<td colspan='2'><input name="submit" type="submit"
					value="Login" /></td>
			</tr>
		</table>
	</form>
</body>
</html>
```

# 测试

1.   访问/admin 需要验证，跳转到登录页面
2.   填写错误的登录名密码，还在该页面，并显示错误提示
3.   填写正确的登录名密码，成功登录，并跳转到admin.html页面
4.   填写权限不够的登录名密码，成功登录，但是提醒权限不够，跳转到error.html页面
5.   直接访问登录页面，登录成功后，跳转到制定页面。


# 根据不同的角色登录成功后跳转不同的页面

>    在实际项目中有这样一种场景，使用同一个登录表单，希望不同的用户登录成功后，显示不同的页面。比如管理员登录成功后，希望直接跳转到后台管理页面，而普通用户显示首页，这个时候可以使用`authentication-success-handler-ref`这个属性配置登录成功后的handler。修改后的form-login属性如下

```xml
<security:form-login 
			login-page="/loginPage"
			authentication-success-handler-ref="loginSuccessHandler"
			default-target-url="/"
			authentication-failure-url="/login?error" 
			username-parameter="username"
			password-parameter="password" />
```

注意：当使用`authentication-success-handler-ref`属性后，`default-target-url`将不再起作用

LoginSuccessHandler.java

```java
package springmvc;

import java.io.IOException;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.security.context.DelegatingApplicationListener;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.context.SecurityContext;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.web.authentication.AuthenticationSuccessHandler;
import org.springframework.security.web.savedrequest.HttpSessionRequestCache;
import org.springframework.security.web.savedrequest.SavedRequest;

public class LoginSuccessHandler implements AuthenticationSuccessHandler {

	public LoginSuccessHandler() {
	}

	/**
	 * 1.获取用户访问的url 2.如果url不为空，验证登录后，直接跳转到该页面 3.如果url为空，则判断用户权限，根据不同的权限跳转到不同的页面
	 */
	public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication auth)
			throws IOException, ServletException {

		SavedRequest saved = new HttpSessionRequestCache().getRequest(request, response);

		String redirectUrl = (saved != null) ? saved.getRedirectUrl() : "";
		if (!redirectUrl.isEmpty()) {
			response.sendRedirect(redirectUrl);
			return;
		}
		// 获取用户登录信息
		Authentication authtication = SecurityContextHolder.getContext().getAuthentication();
		for (GrantedAuthority authority : authtication.getAuthorities()) {
			String role = authority.getAuthority();
			if ("ROLE_ADMIN".equals(role)) {
				redirectUrl = "/admin";
			} else {
				redirectUrl = "/hello";
			}
		}

		response.sendRedirect(redirectUrl);
	}

}

```



# 总结

1.   html和我们自己写的后台controller不是在同一个上下文内的。因此在静态页面访问url的时候，带不带/是有很大区别的，带/是访问host路径，不带/是访问相对路径