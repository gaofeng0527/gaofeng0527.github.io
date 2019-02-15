---
title: springmvc文件上传
date: 2017-11-30 15:28:33
tags: [springmvc]
---

现在基本web应用都需要用到文件上传的功能，不废话，直接上

# 文件上传

这里使用的是CommonsMultipartResolver。

<!-- more -->

## 导入jar

```xml
<dependency>
	<groupId>commons-fileupload</groupId>
	<artifactId>commons-fileupload</artifactId>
	<version>1.3.1</version>
</dependency>
```

## 添加CommonsMultipartResolver配置

```xml
<bean id="multipartResolver"
		class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
   <!-- 设置上传文件的最大值 -->
	<property name="maxUploadSize" value="1048576000"></property>
  	<!-- 多大的文件会被保存在磁盘上 -->
	<property name="maxInMemorySize" value="4096" />
	<property name="defaultEncoding" value="UTF-8"></property>
</bean>
```

## HTML

这里注意一下设置form的enctype属性为multipart/form-data

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html xmlns:th="http://www.thymeleaf.org">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>

<form action="fileUpload" method="post" enctype="multipart/form-data">
	file :<input type="file" name="file">
	<input type="submit">
</form>

</body>
</html>
```

这里如果想上传多个文件的话，只需要多放几个*<input type="file" name="file">*

## controller

```java
package springmvc.controller;

import java.io.File;
import java.io.IOException;
import java.io.UnsupportedEncodingException;
import java.net.URLEncoder;
import java.util.Date;
import java.util.List;


import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.multipart.commons.CommonsMultipartFile;

@Controller
@RequestMapping(value = "/person")
public class PersonController {

	public PersonController() {
		// TODO Auto-generated constructor stub
	}
	
	@RequestMapping(value="/fileUpload" ,method=RequestMethod.GET)
	public String fileUpload() {
		return "person/fileUpload";
	}
	
	
	@RequestMapping(value="/fileUpload" ,method=RequestMethod.POST)
	public String fileUpload(@RequestParam("file") CommonsMultipartFile file) throws IllegalStateException, IOException {
		long starttimes = System.currentTimeMillis();
		System.out.println(file.getOriginalFilename());
		System.out.println(file.getContentType());
		System.out.println(file.getName());
		System.out.println(file.getSize());
		String path="E:/"+(new Date()).getTime()+file.getOriginalFilename();
		File fi = new File(path);
		file.transferTo(fi);
		long endTime =System.currentTimeMillis();
		System.out.println("方法的运行时间："+String.valueOf(endTime-starttimes)+"ms");
		return "person/fileUpload";
	}

}

```

启动运行，没问题

如果是多个文件同时上传，CommonsMultipartFile参数修改为数组即可

>    Javahxjs-j2-gjtx9(jb51.net).rar
>    application/octet-stream    
>    file
>    121780292
>    方法的运行时间：83ms

# 文件上传出现异常的时候

## 自定义一个异常处理类

```java
package springmvc;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.web.multipart.MaxUploadSizeExceededException;
import org.springframework.web.servlet.HandlerExceptionResolver;
import org.springframework.web.servlet.ModelAndView;

/**
 * 自定义异常处理类
 * @author Administrator
 *
 */
public class ExceptionHandler implements HandlerExceptionResolver {

	public ExceptionHandler() {
	}

	public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object obj,
			Exception ep) {

		ModelAndView mview = new ModelAndView();
		if(ep instanceof MaxUploadSizeExceededException) {
			mview.addObject("errormessage", "上传文件大小超出限制");
			mview.setViewName("error");
			return mview;
		}
		return null;
	}

}

```

## 在spring上下文中注册该Handler

```xml
<bean class="springmvc.ExceptionHandler"></bean>
```

## 编写错误页面

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html xmlns:th="http://www.thymeleaf.org">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
<label th:text="${errormessage}">系统出现异常</label>
</body>
</html>
```

运行，搞定