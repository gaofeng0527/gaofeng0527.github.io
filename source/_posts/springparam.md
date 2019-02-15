---
title: Spring接受请求的输入
date: 2018-09-18 10:06:08
tags: springmvc
---

# Spring MVC接受请求的输入

springmvc允许以多种方式将客户端的数据传递到控制器的处理方法中，包括

​	查询参数（Query Parameter）

​	表单参数（Form Parameter）

​	路径变量（Path Variable）

<!-- more -->

# 查询参数

```java
/**
	 * 查询参数
	 * 1.可以处理get或者post请求中的查询参数，例如  query?uname=张高峰&pass=123456;或者post表单中的参数。
	 * 2.使用@RequestParam注解映射参数：当连接中的查询参数和方法中的形参名称不一致时，需要使用name属性说明；required属性表示是否必须。
	 * 3.该注解可以试着默认值
	 * @param uname
	 * @param pass
	 * @param model
	 * @return
	 */
	@RequestMapping(value = "/query", method = RequestMethod.GET)
	public String queryParameter(@RequestParam(name = "uname", defaultValue = "peak", required = false) String uname,
			@RequestParam String pass, Model model) {

		model.addAttribute("uname", uname);
		model.addAttribute("pass", pass);

		return "query";
	}
```

>    http://localhost/spittr/param/query?uname=gaofeng&pass=123 
>
>    页面输出：gaofeng======123

>    当不给uname设定值的时候
>
>    http://localhost/spittr/param/query?uname=&pass=123
>
>    页面输出：peak=====123 //会使用设置的默认值

>http://localhost/spittr/param/query?uname=gaofeng
>
>页面报错：HTTP Status 400 - Required String parameter 'pass' is not present

# 路径参数

```java
/**
	 * 路径参数 1.该种类型的参数也可以使用@RequestParameter注解
	 * 2.建议使用@PathVariable，路径参数需要用大括号括起来，作为占位符；当连接中的路径参数和方法中的形参名称不一致时，需要使用name属性说明
	 * 
	 * @param uid
	 * @param account
	 * @param model
	 * @return
	 */
	@RequestMapping(value = "/pathparam/{userId}/{account}", method = RequestMethod.GET)
	public String pathParameter(@PathVariable(name = "userId") Long uid, @PathVariable String account, Model model) {
		model.addAttribute("uid", uid);
		model.addAttribute("account", account);
		return "pathParam";
	}

    // 还支持正则表达式的方式
    @RequestMapping("/regex/{text:[a-z]+}-{number:\\d+}")
    @ResponseBody
    public String three(@PathVariable String text, @PathVariable Integer number) {
        return "Text: " + text + ", Number: " + number;
    }
  
    // 路径中有 . 时需要用正则，如 http://localhost:8080//question-img/GYYK034C/Aimage002.jpg
    @GetMapping("/question-img/{subjectCode}/{imageName:.+}")
    public void readQuestionImage(@PathVariable String subjectCode, @PathVariable String imageName) {
        ...
    }
```

>    http://localhost/spittr/param/pathparam/123/zhanggf
>
>    页面输出：123=====zhanggf

# 表单数据

```java
/**
	 * 表单数据，可以使用@ModelAttribute注解做映射，匹配页面上相同名称的表单元素，如果有则映射，如果没有则不管。该注解可以省略。
	 * @param spittle
	 * @param model
	 * @return
	 */
	@RequestMapping(value = "/reg", method = RequestMethod.POST)
	public String regSave(@ModelAttribute Spittle spittle, Model model) {
		model.addAttribute("spi", spittle);
		System.out.println(spittle.toString());
		return "redirect:success";
	}
```

```html
	<form method="post">
		message:<input type="text" name="message"></br>
		latitude:<input type="text" name="latitude"></br>
		longitude:<input type="text" name="longitude"></br>
		<button type="submit">reg</button>
	</form>
```

>    Spittle [id=0, message=斯蒂芬斯蒂芬, time=null, latitude=3.0, longitude=4.0]

自动映射了。