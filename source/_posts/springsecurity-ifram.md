---
title: springsecurity-ifram
date: 2018-10-09 16:32:08
tags: [security]
---

# 问题

最近在做项目的时候遇到一个问题，在一个页面中使用到了 ``iframe``引入另外一个页面，结果在访问的时候报错：`in a frame because it set 'X-Frame-Options' to 'deny`

后来上网查找资料得知：

spring Security下，X-Frame-Options默认为DENY,非Spring Security环境下，X-Frame-Options的默认大多也是DENY，这种情况下，浏览器拒绝当前页面加载任何Frame页面，设置含义如下：

    DENY：浏览器拒绝当前页面加载任何Frame页面
    SAMEORIGIN：frame页面的地址只能为同源域名下的页面
    ALLOW-FROM：origin为允许frame加载的页面地址。

<!-- more -->

# 解决方案

在tomcat8以后的版本中，可以通过在web.xml中定义filter设置X-Frame-Options,如下：

```xml
<filter>  
    <filter-name>httpHeaderSecurity</filter-name>  
    <filter-class>org.apache.catalina.filters.HttpHeaderSecurityFilter</filter-class>  
    <init-param>  
        <param-name>antiClickJackingOption</param-name>  
        <param-value>SAMEORIGIN</param-value>  
    </init-param>  
    <async-supported>true</async-supported>  
</filter>  
  
<filter-mapping>  
    <filter-name>httpHeaderSecurity</filter-name>  
    <url-pattern>/*</url-pattern>  
</filter-mapping>  
```



或者修改springsecurity.xml配置文件

```xml
<headers>
	<frame-options policy="SAMEORIGIN"/>
</headers>
```

再次访问，问题解决了。