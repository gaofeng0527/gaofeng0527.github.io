---
title: 绕开过滤器方法和问题
date: 2016-12-12 16:56:06
tags: java
---

# 起因

今天写的是一个在项目中遇到的问题，为了解决XSS攻击，在web.xml配置了一个过滤器，该过滤器会把提交上来的数据进行转义，也就是把html标签给转义了。这就影响到了信息发布的模块，因为在线编辑器存储到服务器上的内容都是转化成html代码存储的，现在过滤器给转义了，所以在显示的时候有出问题了，如图：

<!-- more -->

![](http://ohhigdkar.bkt.clouddn.com/filter1.png)

# 解决方案

1.从数据库取出之后，反转义一次（这个没什么可说的，主要就是字符串操作）

2.绕开上面配置的filter，不对信息发布做拦截（当然可能有点不安全，主要是解决问题时遇到的新问题，值得注意）

## 实例

1.在web.xml配置文件配置filter的时候，可以配置对象实例化需要的参数，这里我们配置的是当在访问某个请求的时候，不过滤该请求，例：

```xml
<filter id="Filter_Xss">
		<filter-name>XSSFilter</filter-name>
		<filter-class>cn.org.ciptc.lms.web.cache.XSSFilter</filter-class>
		<init-param>
			<param-name>exUrl</param-name>
			<param-value>/notice/save</param-value><!--要绕开的请求 -->
		</init-param>
	</filter>
	<filter-mapping id="Filter_Xss__Mapping">
		<filter-name>XSSFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>
```

2.在filter类中，在doFilter方法中需要做这样的判断

```java
public void doFilter(ServletRequest request, ServletResponse response,
			FilterChain paramFilterChain) throws IOException, ServletException {
		String url = ((HttpServletRequest) request).getServletPath();
		String pathUrl = ((HttpServletRequest) request).getPathInfo();
		//System.out.println("url:" + url + " exUrl:" + exUrl + " pathUrl:" + pathUrl);
		if(null != pathUrl && pathUrl.length()>0){
			url = url+pathUrl;
		}
		if (url.equals(exUrl)) {
			paramFilterChain.doFilter(request, response);
		} else {
			//进行过滤操作
		}

	}
```

在这段代码中就有一个坑，在不同的web容器中或者web.xml不同类型的配置中  HttpServletRequest 的getServletPath()方法有可能会返回空，不知道这个问题的同学，可能就发现为什么我在本地测试没问题，部署到服务器上不行了呢，或者我做服务迁移了之后，怎么过滤器失效了呢。那么不妨查询一下是否是这个问题引起的。

这个时候，我们可以调用该类的getPathInfo()方法，如上面的实例，问题就解决了。

