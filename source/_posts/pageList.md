---
title: PageHelper+Layui分页
date: 2017-12-22 13:38:28
tags: [javaweb,springmvc]
---

# 简述

在做项目的时候，我们经常会用到分页的功能。分页又分为物理分页（每次查询固定条数并显示在页面上）和内存分页（全部查出，然后在前端页面做分页，不推荐使用）。

mybitis分页的话还是比较方便的，可以自己编写sql。这里介绍的是一个分页插件`PageHelper`，可以和spring集成使用。

其实分页的思路就是通过前端控制两个参数，一个起始值，然后是每页显示的大小数量。把这两个值传递到后端，编写对应的sql把固定条数的数据查询出来，在前端页面显示。支持首页、下一页、上一页、末页、每页显示数量变化、直接跳转到某一页；另外当我们对列表进行删除或者修改某一条信息的时候，希望仍然停留在当前页。

当然自己写也可以，这里推荐使用LayUi的分页模块。

<!-- more -->

# 引入PageHelper包

```xml
<!-- 分页插件 -->
<dependency>
	<groupId>com.github.pagehelper</groupId>
	<artifactId>pagehelper</artifactId>
	<version>5.1.2</version>
</dependency>
```

# 创建mybitis-config.xml文件

该插件目前支持Oracle,Mysql,MariaDB,SQLite,Hsqldb,PostgreSQL六种数据库分页。

PageHelper的原理是基于拦截器实现的。拦截器的配置有两种方法，一种是在mybatis的配置文件中配置，一种是直接在spring的配置文件中进行，这里只介绍mybitis-config.xml的形式

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<plugins>
		<!-- com.github.pagehelper为PageHelper类所在包名 -->
		<plugin interceptor="com.github.pagehelper.PageInterceptor">
			<!-- 和startPage中的pageNum效果一样 -->
			<property name="offsetAsPageNum" value="true" />
			<!-- 该参数默认为false -->
			<!-- 设置为true时，使用RowBounds分页会进行count查询 -->
			<property name="rowBoundsWithCount" value="true" />
			<!-- 设置为true时，如果pageSize=0或者RowBounds.limit = 0就会查询出全部的结果 -->
			<!-- （相当于没有执行分页查询，但是返回结果仍然是Page类型） -->
			<property name="pageSizeZero" value="true" />
			<!-- 启用合理化时，如果pageNum<1会查询第一页，如果pageNum>pages会查询最后一页 -->
			<!-- 禁用合理化时，如果pageNum<1或pageNum>pages会返回空数据 -->
			<property name="reasonable" value="true" />
			<!-- 3.5.0版本可用 - 为了支持startPage(Object params)方法 -->
			<!-- 增加了一个`params`参数来配置参数映射，用于从Map或ServletRequest中取值 -->
			<!-- 可以配置pageNum,pageSize,count,pageSizeZero,reasonable,orderBy,不配置映射的用默认值 -->
			<!-- 不理解该含义的前提下，不要随便复制该配置 -->
			<property name="params" value="pageNum=start;pageSize=limit;" />
			<!-- 支持通过Mapper接口参数来传递分页参数 -->
			<property name="supportMethodsArguments" value="true" />
			<!-- always总是返回PageInfo类型,check检查返回类型是否为PageInfo,none返回Page -->
			<property name="returnPageInfo" value="check" />
		</plugin>
	</plugins>
</configuration>
```

# 引入mybitis配置文件

```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
	<property name="dataSource" ref="dataSource" />
    <!-- Mapper xml -->
	<property name="mapperLocations" value="classpath:com/gaofeng/mapper/**/*.xml" /> 
	<property name="configLocation" value="classpath:mybitis-config.xml"></property>
</bean>
```

[PageHelper更多用法](https://gitee.com/free/Mybatis_PageHelper)

# 控制层

使用PageHelper不会对sql造成影响，因此service、mapper都不用修改，可以直接在controller中使用

```java
package com.gaofeng.controller;

import java.math.BigDecimal;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;
import java.util.Map;


import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;

import com.gaofeng.pojo.Company;
import com.gaofeng.pojo.Product;
import com.gaofeng.pojo.Result;
import com.gaofeng.pojo.Sysuser;
import com.gaofeng.service.ProductService;
import com.github.pagehelper.PageHelper;
import com.github.pagehelper.PageInfo;

@Controller
@RequestMapping("/home/product")
public class ProductController {

	@Autowired
	private ProductService pservice;

	@GetMapping("/allList")
	public String index(Model model) {
		
		return "product/productList";
	}
	
	@ResponseBody
	@PostMapping("/findProductByPage")
	public PageInfo findProductByPage(int startPage,int pageSize) {
		PageHelper.startPage(startPage, pageSize);
		List<Product> plist = pservice.findAllProduct();
		PageInfo pageInfo = new PageInfo(plist);
		return pageInfo;
	}
	
}

```

因为我前端使用ajax请求获取数据并动态生成表格的，所以这里要在方法上标记`@ResponstBody`注解。

为了前端方便操作分页，直接把pageInfo传递给前端页面（因为我配置了json的处理，前端接收到数据可以直接操作）

下面是教程中给出PageInfo的用法

```java
//获取第1页，10条内容，默认查询总数count
PageHelper.startPage(1, 10);
List`<Country>` list = countryMapper.selectAll();
//用PageInfo对结果进行包装
PageInfo page = new PageInfo(list);
//测试PageInfo全部属性
//PageInfo包含了非常全面的分页属性
assertEquals(1, page.getPageNum());
assertEquals(10, page.getPageSize());
assertEquals(1, page.getStartRow());
assertEquals(10, page.getEndRow());
assertEquals(183, page.getTotal());
assertEquals(19, page.getPages());
assertEquals(1, page.getFirstPage());
assertEquals(8, page.getLastPage());
assertEquals(true, page.isFirstPage());
assertEquals(false, page.isLastPage());
assertEquals(false, page.isHasPreviousPage());
assertEquals(true, page.isHasNextPage());
```

# 前端

productList.html

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html xmlns:th="http://www.thymeleaf.org">
<head>
<meta charset="utf-8">
<title>产品列表--后台管理模板</title>
<meta name="renderer" content="webkit">
<meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
<meta name="viewport"
	content="width=device-width, initial-scale=1, maximum-scale=1">
<meta name="apple-mobile-web-app-status-bar-style" content="black">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="format-detection" content="telephone=no">
<link rel="stylesheet" th:href="@{/static/layui/css/layui.css}"
	href="static/layui/css/layui.css" media="all" />
<link rel="stylesheet"
	th:href="@{/static/css/font_eolqem241z66flxr.css}"
	href="static/css/font_eolqem241z66flxr.css" media="all" />
<link rel="stylesheet" th:href="@{/static/css/news.css}"
	href="static/css/news.css" media="all" />
</head>
<body class="childrenBody">
	<blockquote class="layui-elem-quote news_search">
		<div class="layui-inline">
			<div class="layui-input-inline">
				<input type="text" value="" placeholder="请输入关键字"
					class="layui-input search_input">
			</div>
			<a class="layui-btn search_btn">查询</a>
		</div>
		<div class="layui-inline">
			<a class="layui-btn linksAdd_btn" style="background-color: #5FB878">添加产品</a>
		</div>
		<div class="layui-inline">
			<div class="layui-form-mid layui-word-aux"></div>
		</div>
	</blockquote>
	<div class="layui-form links_list">
		<table class="layui-table">
			<colgroup>
				<col width="50">
				<col width="30%">
				<col>
				<col>
				<col>
				<col>
				<col>
				<col>
				<col width="10%">
			</colgroup>
			<thead>
				<tr>
					<th><input type="checkbox" name="" lay-skin="primary"
						lay-filter="allChoose" id="allChoose"></th>
					<th style="text-align: left;">产品名称</th>
					<th>单价</th>
					<th>区域</th>
					<th>运营商</th>
					<th>产品类型</th>
					<th>状态</th>
					<th>操作</th>
				</tr>
			</thead>
			<tbody class="links_content">
			</tbody>
		</table>
	</div>
	<div id="demo7" style="width: 100%"></div>
	<div id="page"></div>
	<script type="text/javascript" th:src="@{/static/layui/layui.js}"
		src="static/layui/layui.js"></script>
	<script type="text/javascript" th:src="@{/static/js/home/product.js}"
		src="static/js/home/product.js"></script>

</body>
</html>
```

product.js

```js
var startPage = 0;// 起始页
var pageSize = 10;// 每页显示多少条
var total = 0;// 总数
var currPage = 1;// 当前页面
var firstLoad = 0;// 页面首次加载分页

//为了修改后可以直接显示当前页面
function hell(){
	layer.closeAll("iframe");
	getProducts();
	toPage();
}

// ajax 请求后台数据
function getProducts() {
	layui.config({base : "js/"}).use('jquery',function() {
		var $ = layui.jquery;
		$.ajax({
			url : "/gaofeng/home/product/findProductByPage",
			type : "post",
			dataType : "json",
			async : false,
			data : {
				startPage : startPage,
				pageSize : pageSize
			},
			success : function(data) {
				// 渲染页面
				total = data.total;
				startPage = data.pageNum;
				renderDate(data.list);

			}
		})
	})
	
}

function renderDate(plist, curr) {
	layui.config({base : "js/"}).use([ 'form', 'layer', 'jquery', 'laypage' ],function() {
		var form = layui.form();
		var $ = layui.jquery;
		var dataHtml = '';
		if (plist.length != 0) {
			for (var i = 0; i < plist.length; i++) {
				dataHtml += '<tr>'
						+ '<td><input type="checkbox" name="checked" lay-skin="primary" lay-filter="choose"></td>'
						+ '<td align="left">'+ plist[i].title+ '</td>'
						+ '<td>'+ plist[i].price+ '</a></td>'
						+ '<td>'+ plist[i].area+ '</td>'
						+ '<td>'+ plist[i].operator+ '</td>'
						+ '<td>'+ plist[i].ptypeName+ '</td>'
						+ '<td>'+ plist[i].statusName+ '</td>'
						+ '<td>'
						+ '<a class="layui-btn layui-btn-mini links_edit" data-id = "'
						+ plist[i].id
						+ '"><i class="iconfont icon-edit"></i> 编辑</a>'
						+ '<a class="layui-btn layui-btn-danger layui-btn-mini links_del" data-id="'
						+ plist[i].id
						+ '"><i class="layui-icon">&#xe640;</i> 删除</a>'
						+ '</td>'
						+ '</tr>';
			}
		} else {
			dataHtml = '<tr><td colspan="8">暂无数据</td></tr>';
		}
		$(".links_content").html(dataHtml);
		 form.render();//如果不执行这一句，前端的checkbox不显示
	})
	
}
function toPage() {
	layui.config({
		base : "js/"
	}).use(['layer','laypage' ], function() {
		var laypage = layui.laypage;
		laypage.render({
			elem : 'demo7',
			count : total,
			curr : startPage,
			layout : [ 'count', 'prev', 'page', 'next', 'limit', 'skip' ],
			jump : function(obj) {
				console.log(obj);
				console.log(total);
				startPage = obj.curr;
				pageSize = obj.limit;
				if (firstLoad == 1) {//这里是为了防止重复访问（首次）
					getProducts();
				}
				firstLoad = 1;

			}
		});
	})
}

layui.config({base : "js/"}).use([ 'form', 'layer', 'jquery', 'laypage' ],function() {
	var form = layui.form(), 
	layer = layui.layer, 
	laypage = layui.laypage, 
	$ = layui.jquery;
	getProducts();
	toPage();
	// 
	$(".linksAdd_btn").click(
			function() {
				var index = layer.open({
					title : "添加产品",
					type : 2,
					scrollbar : true,
					area : ['100%','100%'],
					content : "productAdd",
					success : function(layero,index) {
						layer.tips('点击此处返回产品列表','.layui-layer-setwin .layui-layer-close',
								{tips : 3});
					}
				})
				// 改变窗口大小时，重置弹窗的高度，防止超出可视区域（如F12调出debug的操作）
				$(window).resize(function() {
					layer.full(index);
				})
				layer.full(index);
		});

	// 全选
	form.on('checkbox(allChoose)',function(data) {
		var child = $(data.elem).parents('table').find('tbody input[type="checkbox"]:not([name="show"])');
		child.each(function(index, item) {
			item.checked = data.elem.checked;
		});
		form.render('checkbox');
	});
	
	// 通过判断产品是否全部选中来确定全选按钮是否选中
	form.on("checkbox(choose)",function(data) {
		var child = $(data.elem).parents('table').find('tbody input[type="checkbox"]:not([name="show"])');
		var childChecked = $(data.elem).parents('table').find('tbody input[type="checkbox"]:not([name="show"]):checked')
		data.elem.checked;
		if (childChecked.length == child.length) {
			$(data.elem).parents('table').find('thead input#allChoose').get(0).checked = true;
		} else {
			$(data.elem).parents('table').find('thead input#allChoose').get(0).checked = false;
		}
		form.render('checkbox');
	})
	
	// 操作
	$("body").on("click",".links_edit",function() { // 编辑
		var _this = $(this);
		var index = layer.open({
			title : "修改产品",
			type : 2,
			content : "productEdit/"+ _this.attr("data-id"),
			success : function(layero,index) {
				
				//layer.tips('点击此处返回产品列表','.layui-layer-setwin .layui-layer-close',{tips : 3});
			}
		});
		
		// 改变窗口大小时，重置弹窗的高度，防止超出可视区域（如F12调出debug的操作）
		$(window).resize(function() {
			layer.full(index);
		})
		layer.full(index);
	})
	
	$("body").on("click",".links_del",function() { // 删除
		var _this = $(this);
		layer.confirm('确定删除此信息？',{icon : 3,title : '提示信息'},
				function(index) {
			$.ajax({
				type : 'GET',
				url : '/gaofeng/home/product/deleteProduct/'+ _this.attr("data-id"),
				dataType : 'json',
				contentType : 'application/json',
				async : true,
				success : function(result) {
					console.log(result.code);
					if (result.code == 0) {
						layer.msg('success:删除成功',{icon : 1,time : 1000});
						setTimeout(function() {
							firstLoad = 0;
							getProducts();
							toPage();
						}, 800);
					} else {
						layer.msg('eles:删除失败！'+ result.message,{icon : 2,time : 1000});
					}
				},error : function(result,type) {
					layer.msg('error:删除失败！',{icon : 2,time : 1000});
				}
			});
			layer.close(index);
		});
	})
})
```

# 总结

当我们修改一条信息后，希望刷新数据，但是依然停留在当前页面。在修改信息的页面，当数据保存成功以后，可以调用父类的方法，进行页面刷新并关闭页面。**当然这是我想到的方法（因为使用layui框架，都是ajax请求的数据，没有做页面整体刷新），可能会有更好的方法。**




































