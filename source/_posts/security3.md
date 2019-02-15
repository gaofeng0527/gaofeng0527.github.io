---
title: Security 配置用户数据源
date: 2017-12-08 10:53:32
tags: [security]
---

# 背景

前面我们已经初步了解了Spring Security安全框架，并可以简单使用。但是之前的用户信息我们都是写在配置文件中。如果用户数量比较多了，或者用户信息经常变化，在写到配置文件中就不太现实，下面就讲一下用其他数据源来存储用户信息的情况。

<!-- more -->

# 模拟一下数据库

接口

```java
package springmvc.service;

import springmvc.pojo.User;
import springmvc.pojo.UserSecurity;

public interface UserService {
	
	void save(User user);
	
	UserSecurity findByName(String name);

}

```



实现类

```java
package springmvc.service.impl;

import java.util.HashMap;
import java.util.Map;

import org.springframework.stereotype.Service;

import springmvc.pojo.User;
import springmvc.pojo.UserSecurity;
import springmvc.service.UserService;

@Service
public class UserServiceImpl implements UserService {
	private static Map<String, UserSecurity> users = new HashMap<String, UserSecurity>();
	static {
		// 模拟数据源，可以是多种，如数据库，LDAP，从配置文件读取等
		users.put("admin", new UserSecurity("admin", "admin", "ROLE_ADMIN"));
		users.put("alice", new UserSecurity("alice", "admin", "ROLE_USER"));
	}

	public UserServiceImpl() {
		// TODO Auto-generated constructor stub
	}

	public void save(User user) {

		System.out.println(user.toString());
	}

	public UserSecurity findByName(String name) {
		// TODO Auto-generated method stub
		return users.get(name);
	}
}

```

# 用户对象类 

用户类需要继承org.springframework.security.core.userdetails.User

```java
package springmvc.pojo;

import java.util.Collection;
import java.util.HashSet;

import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.AuthorityUtils;

import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class UserSecurity extends org.springframework.security.core.userdetails.User {

	private Long id;
	private String username;
	private String password;

	public UserSecurity() {
		// 父类不允许空的用户名、密码和权限，所以给个默认的，这样就可以用默认的构造函数创建 User 对象。
		super("non-exist-username", "", (Collection<? extends GrantedAuthority>) new HashSet<Object>());
	}
	
	/**
	 * 
	 * @param username 帐号
	 * @param password 密码
	 * @param roles 角色
	 */
	public UserSecurity(String username,String password,String...roles) {
		this(username,password,true,roles);
	}

	/**
	 * 
	 * @param username 帐号
	 * @param password 密码
	 * @param enabled 是否禁用
	 * @param roles 角色
	 */
	public UserSecurity(String username, String password, boolean enabled, String... roles) {
		super(username, password, enabled, true, true, true, AuthorityUtils.createAuthorityList(roles));
		this.username = username;
		this.password = password;

	}

}

```

# UserDetailsService

UserDetailsService的作用是根据登录表单中的用户名查找用户信息用于身份验证。Spring Security中验证分为两部：1.身份验证  2.权限验证

```java
package springmvc;

import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;

import springmvc.pojo.UserSecurity;
import springmvc.service.UserService;
import springmvc.service.impl.UserServiceImpl;

public class UserSecurityService implements UserDetailsService {

	private UserService us = new UserServiceImpl();
	public UserSecurityService() {
		// TODO Auto-generated constructor stub
	}

	/**
	 * 根据用户提交的username查找用户信息，如密码，权限等信息
	 * Spring Security根据查找到的用户密码和加密后的用户输入的密码进行比较，用来身份验证
	 * 然后使用用户的角色和页面访问需要的角色对比，如果成功则验证成功。
	 */
	public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
		// TODO Auto-generated method stub
		UserSecurity user = us.findByName(username);
		if(null == user) {
			 throw new UsernameNotFoundException(username + " not found!");
		}
		return user;
	}

}

```

# 配置用户信息来源（spring-security.xml）

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
		<security:intercept-url pattern="/admin"
			access="ROLE_ADMIN" />
		<!-- 配置登录验证的一些信息 1.login-page:get请求登录页面的url 2.login-processing-url：post请求，登录页面提交的请求（点击登录后，表单提交的信息会被security默认的登录验证拦截器拦截，并 
			判断用户信息，处理完之后，才会进入到这个地方配置的url对应的controller方法中。） 3.default-target-url:当用户直接访问登录页面，登录成功后，跳转到该属性配置的url 
			4.配置了authentication-successs-handler-ref 是登录验证成功后，跳转的handler，default-target-url会失效 -->
		<security:form-login login-page="/loginPage"
			authentication-success-handler-ref="loginSuccessHandler"
			default-target-url="/" authentication-failure-url="/login?error"
			username-parameter="username" password-parameter="password" />
		<!-- 配置注销登录的一些配置 1.logout-url：get请求，注销的url 2.logout-success-url:get请求，注销成功后访问的url -->
		<security:logout logout-url="/logout"
			logout-success-url="/login?logout" />
		<!-- 登录成功，却没有权限的页面 error-page:没有权限访问的url -->
		<security:access-denied-handler
			error-page="/deny" />
		<security:csrf disabled="true" />
	</security:http>
    <!-- user-service-ref 配置用户数据来源信息 -->
	<security:authentication-manager>
		<security:authentication-provider
			user-service-ref="userSecurityService">
		</security:authentication-provider>
	</security:authentication-manager>
	<bean id="loginSuccessHandler" class="springmvc.LoginSuccessHandler"></bean>
	<bean id="userSecurityService" class="springmvc.UserSecurityService"></bean>
</beans>

```

# Spring Security加密密码

我们在存储用户数据的时候，肯定不会存储明文的密码。这个时候，我们可以配置security的密码加密方式（要和存储密码时的加密方式一致）。在`<security-authentication-provider>`标签内添加`<security:password-encoder hash="bcrypt">`标签。bcrypt是一中加密方式：

>    BCrypt 算法与 MD5/SHA 算法有一个很大的区别，每次生成的 hash 值都是不同的，就可以免除存储 salt，暴力破解起来也更困难。BCrypt 加密后的字符长度比较长，有60位，所以用户表中密码字段的长度，如果打算采用 BCrypt 加密存储，字段长度不得低于 68(需要前缀 `{bcrypt}`)。（彪哥）
>
>    

```java
package springmvc.test;

import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

public class PasswordEncode {

	public static void main(String[] args) {

		PasswordEncoder pe = new BCryptPasswordEncoder();
		String crypt = pe.encode("zhanggaofeng");
		System.out.println(crypt);
		for (int i = 0; i < 5; i++) {
			System.out.println(pe.encode("zhanggaofeng"));
			System.out.println(pe.matches("zhanggaofeng", crypt));
		}
	}

}

```

上面那个是bcrypt的使用方式

修改后的spring-security.xml配置文件如下

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
		<security:intercept-url pattern="/admin"
			access="ROLE_ADMIN" />
		<!-- 配置登录验证的一些信息 1.login-page:get请求登录页面的url 2.login-processing-url：post请求，登录页面提交的请求（点击登录后，表单提交的信息会被security默认的登录验证拦截器拦截，并 
			判断用户信息，处理完之后，才会进入到这个地方配置的url对应的controller方法中。） 3.default-target-url:当用户直接访问登录页面，登录成功后，跳转到该属性配置的url 
			4.配置了authentication-successs-handler-ref 是登录验证成功后，跳转的handler，default-target-url会失效 -->
		<security:form-login login-page="/loginPage"
			authentication-success-handler-ref="loginSuccessHandler"
			default-target-url="/" authentication-failure-url="/login?error"
			username-parameter="username" password-parameter="password" />
		<!-- 配置注销登录的一些配置 1.logout-url：get请求，注销的url 2.logout-success-url:get请求，注销成功后访问的url -->
		<security:logout logout-url="/logout"
			logout-success-url="/login?logout" />
		<!-- 登录成功，却没有权限的页面 error-page:没有权限访问的url -->
		<security:access-denied-handler
			error-page="/deny" />
		<security:csrf disabled="true" />
	</security:http>
	<security:authentication-manager>
		<security:authentication-provider
			user-service-ref="userSecurityService">
			<security:password-encoder hash="bcrypt"></security:password-encoder>
		</security:authentication-provider>
	</security:authentication-manager>
	<bean id="loginSuccessHandler" class="springmvc.LoginSuccessHandler"></bean>
	<bean id="userSecurityService" class="springmvc.UserSecurityService"></bean>
</beans>

```

`<security:password-encoder hash="bcrypt"></security:password-encoder>`

在数据源中存储的也应该是加密后的密码：

``` java
static {
		// 模拟数据源，可以是多种，如数据库，LDAP，从配置文件读取等
		users.put("zhanggaofeng", new UserSecurity("zhanggaofeng", "$2a$10$hWA4T4Zwxiz0TSg39XKFuOLe9tJxak2cCIYWKyjxv5SCXl/qnMvJG", "ROLE_ADMIN"));
		users.put("alice", new UserSecurity("alice", "admin", "ROLE_USER"));
	}
```



# 测试

没有问题，具体的设计还要根据实际需求进行开发。