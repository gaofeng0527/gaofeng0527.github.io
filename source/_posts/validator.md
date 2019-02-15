---
title: springmvc数据有效验证（validator）
date: 2017-11-29 17:18:15
tags: [springmvc]
---

# 开篇

数据有效性验证，就是保证从前台传输到后台的数据是有效的正确数据。因此前端和后台都要添加数据的有效性验证。springmvc对数据的有效性验证有很好的支持。

<!-- more -->

# spring validator接口验证

## 导入需要的jar

这里使用maven

```xml
<dependency>
	<groupId>javax.validation</groupId>
	<artifactId>validation-api</artifactId>
	<version>2.0.0.Final</version>
</dependency>
```



## 需要一个实体类

```java
package springmvc.pojo;
import javax.validation.constraints.NotNull;
import javax.validation.constraints.Size;

public class User {
	private String uname;
	private String mome;
	public User() {
		// TODO Auto-generated constructor stub
	}
	//get set。。。。

```

## 需要一个验证类

验证类需要实现org.springframework.validation.Valdator接口，并重写supports()和validate()方法

```java
package springmvc;

import org.springframework.validation.Errors;
import org.springframework.validation.Validator;

import springmvc.pojo.User;

public class UserValidator implements Validator {

	public UserValidator() {
	}

	/**
	 * 确定该验证类只对哪个实体类有效
	 */
	public boolean supports(Class<?> clazz) {
		return User.class.equals(clazz);
	}

	/**
	 * 实际验证操作
	 */
	public void validate(Object obj, Errors error) {
		User user = (User)obj;
		if(null == user.getUname() || "".equals(user.getUname())) {
			error.reject("uname", null, "uname is empty");
		}
	}

}

```

## 使用该验证类

准备工作已经做好，下面看一下怎么在controller中对需要验证的类验证呢

```java
package springmvc.controller;

import java.io.UnsupportedEncodingException;
import java.net.URLEncoder;
import java.util.List;

import javax.validation.Valid;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.BindingResult;
import org.springframework.validation.DataBinder;
import org.springframework.validation.ObjectError;
import org.springframework.web.bind.annotation.InitBinder;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

import springmvc.UserValidator;
import springmvc.pojo.User;

@Controller
@RequestMapping(value = "/person")
public class PersonController {

	public PersonController() {
		// TODO Auto-generated constructor stub
	}
	
  	/**
	 * 可以通过DataBinder来指定需要使用的Validator，
	 * 可以通过@InitBinder标记的方法initBinder设置当前Controller需要使用的Validator是UserValidator。
	 * 
	 * @param binder
	 */
	@InitBinder
	public void initBinder(DataBinder binder) {
		binder.setValidator(new UserValidator());
	}

	/**
	 * 1.需要验证的对象需要用@Valid进行标注的
	 * Validation对它的实现。
	 * 2.处理器方法必须给定包含Errors的参数，这可以是Errors本身，也可以是它的子类BindingResult，使用了Errors参数就是告诉
	 * Spring关于表单对象数据校验的错误将由我们自己来处理，否则Spring会直接抛出异常
	 * 3.Errors这个参数是必须紧挨着@Valid参数的，即必须紧挨着需要校验的参数，这就意味着我们有多少个@Valid参数就需要有多少个对应的Errors参数，它们是一一对应的。
	 * @param model
	 * @return
	 */
	@RequestMapping(value = "register", method = RequestMethod.POST)
	public String register(@Valid User user, BindingResult result, Model model) throws UnsupportedEncodingException {
		if (result.hasErrors()) {
			List<ObjectError> list = result.getAllErrors();
			for (ObjectError e : list) {
				System.out.println(e.getDefaultMessage());
			}
			return register(model);
		}
		System.out.println(user.toString());
		return "redirect:/person/model/" + URLEncoder.encode(user.getUname(), "UTF-8") + "/"
				+ URLEncoder.encode(user.getMome(), "utf-8");
	}
	
}
```

以上就是接口验证方式，这种方式UserValidator验证类只对PersonController有效。如果想在多个controller中都能使用该验证类，需要在springmvc配置文件中添加bean

```xml
<mvc:annotation-driven validator="userValidator"/>
<bean id="userValidator" class="....UserValidator"></bean>
```

# 使用注解进行验证

## 导入要用的jar

 JSR-303是一个数据验证的规范,当我们在SpringMVC中需要使用到JSR-303的时候就需要我们提供一个对JSR-303规范的实现，Hibernate Validator是实现了这一规范的。JSR-303的校验是基于注解的，它内部已经定义好了一系列的限制注解，我们只需要把这些注解标记在需要验证的实体类的属性上或是其对应的get方法上。这里需要到如Hibernate Validator 的jar包，这个是和ORM框架无关的。

```xml
<!-- 数据校验 -->
<dependency>
	<groupId>javax.validation</groupId>
	<artifactId>validation-api</artifactId>
	<version>2.0.0.Final</version>
</dependency>
<dependency>
	<groupId>org.hibernate.validator</groupId>
	<artifactId>hibernate-validator</artifactId>
	<version>6.0.2.Final</version>
</dependency>
```

## 修改之前的User实体类

```java
package springmvc.pojo;
import javax.validation.constraints.NotNull;
import javax.validation.constraints.Size;
public class User {
	@NotNull(message = "用户名不能为空")
	@Size(min = 6, max = 30)
	private String uname;

	private String mome;

	public User() {
		// TODO Auto-generated constructor stub
	}
	//get set....
}

```



## 开启注解扫描

这里要注意，在springmvc配置文件中要开启 <mvc:annotation-driven/>

我使用的是springmvc 4.2.6，我感觉4.0以后的版本这个配置都行。

><mvc:annotation-driven /> 是一种简写形式，完全可以手动配置替代这种简写形式，简写形式可以让初学都快速应用默认配置方案。<mvc:annotation-driven /> 会自动注册DefaultAnnotationHandlerMapping与AnnotationMethodHandlerAdapter 两个bean,是spring MVC为@Controllers分发请求所必须的。
>并提供了：数据绑定支持，@NumberFormatannotation支持，@DateTimeFormat支持，@Valid支持，读写XML的支持（JAXB），读写JSON的支持（Jackson）。

```xml
<!-- 在springmvc配置文件中添加以下配置 -->
<mvc:annotation-driven/>
```

## controller

这里就不需要

```java
@InitBinder
public void initBinder(DataBinder binder) {
	binder.setValidator(new UserValidator());
}
```



```java
package springmvc.controller;

import java.io.UnsupportedEncodingException;
import java.net.URLEncoder;
import java.util.List;

import javax.validation.Valid;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.BindingResult;
import org.springframework.validation.DataBinder;
import org.springframework.validation.ObjectError;
import org.springframework.web.bind.annotation.InitBinder;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

import springmvc.UserValidator;
import springmvc.pojo.User;

@Controller
@RequestMapping(value = "/person")
public class PersonController {

	public PersonController() {
		// TODO Auto-generated constructor stub
	}
	/**
	 * 1.需要验证的对象需要用@Valid进行标注的
	 * Validation对它的实现。
	 * 2.处理器方法必须给定包含Errors的参数，这可以是Errors本身，也可以是它的子类BindingResult，使用了Errors参数就是告诉
	 * Spring关于表单对象数据校验的错误将由我们自己来处理，否则Spring会直接抛出异常
	 * 3.Errors这个参数是必须紧挨着@Valid参数的，即必须紧挨着需要校验的参数，这就意味着我们有多少个@Valid参数就需要有多少个对应的Errors参数，它们是一一对应的。
	 * @param model
	 * @return
	 */
	@RequestMapping(value = "register", method = RequestMethod.POST)
	public String register(@Valid User user, BindingResult result, Model model) throws UnsupportedEncodingException {
		if (result.hasErrors()) {
			List<ObjectError> list = result.getAllErrors();
			for (ObjectError e : list) {
				System.out.println(e.getDefaultMessage());
			}
			return register(model);
		}
		System.out.println(user.toString());
		return "redirect:/person/model/" + URLEncoder.encode(user.getUname(), "UTF-8") + "/"
				+ URLEncoder.encode(user.getMome(), "utf-8");
	}
	
}
```

# 分组验证

校验规则是在pojo制定的，而同一个pojo可以被多个Controller使用，此时会有问题，即：不同的Controller方法对同一个pojo进行校验，此时这些校验信息是共享在这不同的Controller方法中，但是实际上每个Controller方法可能需要不同的校验，在这种情况下，就需要使用分组校验来解决这种问题，通俗的讲，一个pojo中有很多属性，controller中的方法1可能只需要校验pojo中的属性1，controller中的方法2只需要校验pojo中的属性2，但是pojo中的校验注解有很多，怎样才能使方法1只校验属性1，方法二只校验属性2呢？就需要用分组校验来解决了。

## 创建两个空接口

```java
package springmvc.validator;

/**
 * 分组校验
 * @author Administrator
 *
 */
public interface IValidator1 {

}
```

```java
package springmvc.validator;

public interface IValidator2 {

}
```

## 修改实体类

在验证注解中，可以设置groups属性，设置不同的接口作为分组

```java
package springmvc.pojo;

import javax.validation.constraints.NotBlank;
import javax.validation.constraints.Size;

import springmvc.validator.IValidator1;
import springmvc.validator.IValidator2;

public class User {

	private Long id;

	@NotBlank(message = "用户名不能为空",groups= {IValidator2.class})
	@Size(min = 6, max = 30,groups= {IValidator1.class})
	private String uname;

	private String mome;

	public User() {
		// TODO Auto-generated constructor stub
	}
  //get  set

}

```

## 在controller中也使用groups属性

```java
@RequestMapping(value = "register", method = RequestMethod.POST)
	public String register(@Validated(value= {IValidator2.class}) User user, BindingResult result, Model model) throws UnsupportedEncodingException {
		if (result.hasErrors()) {
			List<ObjectError> list = result.getAllErrors();
			for (ObjectError e : list) {
				System.out.println(e.getDefaultMessage());
			}
			return register(model);
		}
		System.out.println(user.toString());
		return "redirect:/person/model/" + URLEncoder.encode(user.getUname(), "UTF-8") + "/"
				+ URLEncoder.encode(user.getMome(), "utf-8");
	}
```



# 总结

spring版本有很多，有些配置也是不一样的，大家在查看资料的时候，要注意。校验基本上就讲到这，后面在研究一下自定义校验。





