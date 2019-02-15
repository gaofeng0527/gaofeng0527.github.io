---
title: thymeleaf标签属性的实际演示
date: 2017-12-01 13:18:36
tags: [thymeleaf]
---

Thymeleaf的主要目标在于提供一种可被浏览器正确显示的、格式良好的模板创建方式，因此也可以用作静态建模。Thymeleaf的模板还可以用作工作原型，Thymeleaf会在运行期替换掉静态值。

# bean值替换

当该html作为静态资源的时候，显示*张高峰*，在运行期间会解析*${uname}*的值，如果uname没有值的话，会显示空白

<!-- more -->

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"
	xmlns:th="http://www.thymeleaf.org">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
<label th:text="${uname}">张高峰</label>
</body>
</html>
```

# 简单数据替换(数字，日期)

```html

<!-- 静态资源显示：180；运行时显示： 180.00-->
价格：<label th:text="${#numbers.formatDecimal(price,1,2)}">180</label><br>
<!-- 静态资源显示：2017/12/01；运行时显示：2017-12-01 11:49:35-->
日期：<label th:text="${#dates.format(times,'yyyy-MM-dd HH:mm:ss')}">2017/12/01</label>

```



# 迭代

```html
<table>
		<thead>
			<tr>
				<td>编号</td>
				<td>姓名</td>
				<td>二货</td>
			</tr>
		</thead>
		<tbody th:remove="all-but-first">
			<tr th:each="user:${ulist}">
				<td th:text="${user.id}">id</td>
				<td th:text="${user.uname}">name</td>
				<td th:text="${user.mome}">mome</td>
			</tr>
			<tr>
				<td>s</td>
				<td>e</td>
				<td>r5</td>
			</tr>
		</tbody>
	</table>
```

# th:text和th:utext

前面已经多次使用th:text属性，这个是为了显示文本的。但是当我们要显示的文本中出现一些特殊符号的时候，就会被转义例如：**Welcome to our <b>fantastic</b> grocery store!**

如果使用th:text 那么显示出来的就会是：

>    Welcome to our <b>fantastic</b> grocery store!

我们并不希望浏览器给我们转义,这个时候就需要用到th:utext了

```html
<p th:utext="#{home.welcome}">Welcome to our grocery store!</p>
```

>    <p>Welcome to our <b>fantastic</b> grocery store!</p>

# 简单表达式

>    1.   变量表达式：${...} 和springEl表达式基本相同
>    2.   选择变量表达式：*{...} 结合th:object属性使用
>    3.   消息表达式：#{...} 用来读取properties文件
>    4.   链接网址表达式：@{...}
>    5.   片段表达式：~{...}

```html
<div th:object="${session.user}">
<p>Name: <span th:text="*{firstName}">Sebastian</span>
.</p>
<p>Surname: <span th:text="*{lastName}">Pepper</span>.
</p>
<p>Nationality: <span th:text="*{nationality}">Saturn<
  /span>.</p>
</div>
```

```html
<div>
<p>Name: <span th:text="${session.user.firstName}">Sebas
tian</span>.</p>
<p>Surname: <span th:text="${session.user.lastName}">Pep
per</span>.</p>
<p>Nationality: <span th:text="${session.user.nationalit
y}">Saturn</span>.</p>
</div>
```

以上两段代码效果相同，这两种表达式也可以混合使用

# 比较和相等运算符

>    1.   比较运算符:>,<,>=,<=(gt,lt,ge,le)
>    2.   相等运算符：==,!=(eq,ne)

# 