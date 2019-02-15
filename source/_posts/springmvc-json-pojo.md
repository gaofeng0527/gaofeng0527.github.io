---
title: springmvc+thymeleaf+layui表单提交以及json对象与实体对象的映射
date: 2017-12-21 09:32:03
tags: [springmvc,thymeleaf]
---

# 背景

最近自己在项目的时候遇到了一个问题，视图解析使用的是`thymeleaf`，前端框架使用了`layui`。在做表单提交的时候在传输json对象的时候，到后端的时候实体映射没有成功。后来测试发现`JSON.stringify` 和`Content-Type`的关系。

<!-- more -->

# FastJson

我使用的是`FastJson`操作json对象

pom.xml

```xml
<dependency>
	<groupId>com.alibaba</groupId>
	<artifactId>fastjson</artifactId>
	<version>1.2.41</version>
</dependency>
```

spring-servlet.xml

```xml
<!-- 注解映射支持 -->
<mvc:annotation-driven>
	<mvc:message-converters>
	<!-- StringHttpMessageConverter 编码为UTF-8，防止乱码 -->
		<bean class="org.springframework.http.converter.StringHttpMessageConverter">
			<constructor-arg value="UTF-8" />
			<property name="supportedMediaTypes">
				<list>
					<bean class="org.springframework.http.MediaType">
						<constructor-arg index="0" value="text" />
						<constructor-arg index="1" value="plain" />
						<constructor-arg index="2" value="UTF-8" />
					</bean>
					<bean class="org.springframework.http.MediaType">
						<constructor-arg index="0" value="*" />
						<constructor-arg index="1" value="*" />
						<constructor-arg index="2" value="UTF-8" />
					</bean>
				</list>
			</property>
		</bean>
		<!-- FastJson -->
		<bean class="com.alibaba.fastjson.support.spring.FastJsonHttpMessageConverter4">
			<property name="supportedMediaTypes">
				<list>
					<value>text/html;charset=UTF-8</value>
					<value>application/json;charset=UTF-8</value>
				</list>
			</property>
			<property name="fastJsonConfig">
				<bean class="com.alibaba.fastjson.support.config.FastJsonConfig">
					<property name="features">
						<list>
							<value>AllowArbitraryCommas</value>
							<value>AllowUnQuotedFieldNames</value>
							<value>DisableCircularReferenceDetect</value>
						</list>
					</property>
					<property name="dateFormat" value="yyyy-MM-dd HH:mm:ss" />
				</bean>
			</property>
		</bean>
	</mvc:message-converters>
</mvc:annotation-driven>
```

上面的配置是处理乱码，和处理json映射的。

# Form表单

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html xmlns:th="http://www.thymeleaf.org">
<head>
<meta charset="utf-8">
<title>产品修改</title>
<meta name="renderer" content="webkit">
<meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
<meta name="viewport"
	content="width=device-width, initial-scale=1, maximum-scale=1">
<meta name="apple-mobile-web-app-status-bar-style" content="black">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="format-detection" content="telephone=no">
<link rel="stylesheet" th:href="@{/static/layui/css/layui.css}"
	href="/static/layui/css/layui.css" media="all" />
<link rel="stylesheet"
	th:href="@{/static/css/font_eolqem241z66flxr.css}"
	href="/static/css/font_eolqem241z66flxr.css" media="all" />
</head>
<body class="childrenBody">
	<form class="layui-form" style="width: 80%;">
		<div class="layui-form-item">
			<label class="layui-form-label">产品名称</label>
			<div class="layui-input-block">
				<input type="text" name="title" th:value="${product.title}" class="layui-input" lay-verify="required" placeholder="请输入产品名称">
				<input type="hidden" name="id" th:value="${product.id}">
			</div>
		</div>
		<div class="layui-form-item">
			<label class="layui-form-label">单价</label>
			<div class="layui-input-block">
				<input type="text" name="price" th:value="${product.price}" class="layui-input" lay-verify="required|number" placeholder="请输入产品单价">
			</div>
		</div>
		<div class="layui-form-item">
			<label class="layui-form-label">优惠金额</label>
			<div class="layui-input-block">
				<input type="text" class="layui-input" name="dcoupon" th:value="${product.dcoupon}" lay-verify="required|number" placeholder="请输入优惠金额">
			</div>
		</div>
		<div class="layui-form-item">
			<label class="layui-form-label">区域</label>
			<div class="layui-input-block">
				<select name="area" lay-filter="aihao" th:value="${product.area}">
					<option value="东城区">东城区</option>
					<option value="西城区">西城区</option>
					<option value="朝阳区" selected="">朝阳区</option>
					<option value="大兴区">大兴区</option>
					<option value="丰台区">丰台区</option>
					<option value="海淀区">海淀区</option>
				</select>
			</div>
		</div>
		<div class="layui-form-item">
			<label class="layui-form-label">运营商</label>
			<div class="layui-input-block">
				<select name="operator" lay-filter="aihao" th:value="product.operator">
					<option value="移动" th:selected="${product.operator=='移动'}" selected="">移动</option>
					<option value="联通" th:selected="${product.operator=='联通'}">联通</option>
					<option value="电信" th:selected="${product.operator=='电信'}">电信</option>
				</select>
			</div>
		</div>
		<div class="layui-form-item">
			<label class="layui-form-label">产品类型</label>
			<div class="layui-input-block">
				<select name="type" lay-filter="aihao" th:value="product.type">
					<option value="1" th:selected="${product.type==1}" selected="">年付</option>
					<option value="2" th:selected="${product.type==2}">季付</option>
					<option value="3" th:selected="${product.type==3}">月付</option>
				</select>
			</div>
		</div>
		<div class="layui-form-item">
			<label class="layui-form-label">产品状态</label>
			<div class="layui-input-block">
				<select name="status" lay-filter="aihao" th:value="product.status">
					<option value="0" th:selected="${product.status==0}" selected="">在用</option>
					<option value="1" th:selected="${product.status==1}">弃用</option>
				</select>
			</div>
		</div>
		
		<div class="layui-form-item">
			<label class="layui-form-label">产品描述</label>
			<div class="layui-input-block">
				<textarea placeholder="请输入产品描述" class="layui-textarea" name="discounts" th:text="${product.discounts}"></textarea>
			</div>
		</div>
		<div class="layui-form-item">
			<div class="layui-input-block">
				<button class="layui-btn" lay-submit="" lay-filter="addProduct">立即提交</button>
				<button type="reset" class="layui-btn layui-btn-primary">重置</button>
			</div>
		</div>
	</form>
	<script type="text/javascript" th:src="@{/static/layui/layui.js}"
		src="static/layui/layui.js"></script>
	<script type="text/javascript"
		th:src="@{/static/js/home/productEdit.js}" src="linksAdd.js"></script>
</body>
</html>
```

```js
layui.config({
			base : "js/"
		}).use([ 'form', 'layer', 'jquery', 'layedit', 'laydate' ],
				function() {
					var form = layui.form(), 
						layer = layui.layer, 
						laypage = layui.laypage, 
						layedit = layui.layedit, 
						laydate = layui.laydate, 
						$ = layui.jquery;
					form.on("submit(addProduct)", function(data) {
						$.ajax({
							type : 'POST',
							url : '/gaofeng/home/product/productEdit',
							dataType : 'json',
							data : data.field,
							async : true,
							success : function(result) {
								console.log(result.code);
								if (result.code == 0) {
									layer.msg('success:保存成功', {
										icon : 1,
										time : 1000
									});
									setTimeout(function() {
										layer.closeAll("iframe");
								 		//刷新父页面
								 		parent.location.reload();
									}, 800);
								} else {
									consol.alert(result.code);
									layer.msg('eles:保存失败！' + result.msg, {
										icon : 2,
										time : 1000
									});
								}
							},
							error : function(result, type) {
								layer.msg('error:保存失败！', {
									icon : 2,
									time : 1000
								});
							}
						});
						console.log(data.field)
						return false;
					})

				})

```

# controller

```java
@PostMapping("/productEdit")
@ResponseBody
public Result productEdit(Product product) {
	pservice.updateProduct(product);
	Result result = new Result();
	result.setSuccess(true);
	result.setMessage("保存成功");
	result.setCode(0);
	return result;
}
```

Result.java

```java
package com.gaofeng.pojo;

public class Result {

	private boolean success;
	private String message;
	private Object data;
	private int code;
	//省略get，set方法
}

```

>    如果处理ajax请求的时候，在方法上必须添加@ResponseBody注解，使用@Responsebody标识的方法表示该方法的返回结果直接写入HTTP response body中，一般在异步获取数据时使用，在使用@RequestMapping后，返回值通常解析为跳转路径，加上@Responsebody后返回结果不会被解析为跳转路径，而是直接写入HTTP response body中。比如**异步获取json数据，加上@Responsebody后，会直接返回json数据。**

# 总结

当我们在使用ajax向后端传输数据的时候`data : data.field`相当于直接传输json对象，经过FastJson会自动给我们封装成对象。而当我们使用了`data : JSON.stringify(data.field)`方法后，必须设置contentType: `application/json`属性。而且后端在接收实体对象的时候前面要加上@RequestBody注解



​	