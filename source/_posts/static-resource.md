---
title: springmvc对静态资源的处理
date: 2017-12-01 10:21:22
tags: [springmvc]
---

# 开场白

不知道为啥，写笔记的时候总是喜欢写开场白，有点自言自语的感觉，哈哈哈。

初学springmvc的时候，遇到一个坑，就是在页面引用**js**或者**css**文件的时候总是失败，后来上网查资料才明白了原因。

在使用springmvc的时候，我们首先要在web.xml中配置**DispatcherServlet**，并且拦截所有URL请求

```xml
<servlet>
	<servlet-name>springDispatcherServlet</servlet-name>
	<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
	<init-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath:springmvc.xml</param-value>
	</init-param>
	<load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
	<servlet-name>springDispatcherServlet</servlet-name>
	<url-pattern>/</url-pattern>
</servlet-mapping>
```

这样的话.js、.css、.jpg等文件的请求也都被DispatcherServlet给拦截了，可是又没有相应的处理方法，因此也就访问不到了。

<!-- more -->

# 修改springmvx.xml

springmvc给了很好的解决办法，这里就介绍一中我感觉最好用的办法<mvc:resource/>标签，在springmvc.xml配置文件中添加已下配置

```xml
<mvc:resources location="/resources/" mapping="/resources/**"></mvc:resources>
<mvc:resources location="/js/" mapping="/js/**"></mvc:resources>
<mvc:resources location="/css/" mapping="/css/**"></mvc:resources>
```

>    location:webapp根目录，如果要放到别的目录的话需要添加目录名称，比如要放到WEB-INF目录下，location="/WEB-INF/js/" 
>
>    mapping:这里的**表示该目录下的所有目录，包括子目录



# html

```html
<link href="/css/style.css" th:href="@{/css/style.css}">
```

就可以了