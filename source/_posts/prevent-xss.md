---
title: 防止XSS攻击
date: 2019-05-31 16:10:09
tags: [javaweb]
---

# XSS攻击

XSS攻击全称跨站脚本攻击，是一种在`web`应用中的计算机安全漏洞，它允许恶意web用户将代码植入到提供给其他用户使用的页面中。例如：不法用户在评论某条博客时，植入类似`<script>alert('一个大傻瓜')</script>`可执行的js脚本保存到服务器，当在显示的时候，就会出现弹出框，这是最简单的XSS攻击。

>    服务器处理富文本编辑器提交的内容时, 因排版的需求不能对 HTML 标签进行转义, 但为了防止 XSS 攻击, 又必须过滤掉其中的 JS 代码, 在 Java 中使用 Jsoup 正好可以满足此要求
>
>    

[参考](https://www.jianshu.com/p/32abc12a175a)

# 解决思路

1.在前端页面，可以通过js过滤一下特殊字符。

2.在服务器端通过过滤器，拦截请求，过滤参数，清除危险字符

<!-- more -->

# 实例

## 导入所需jar

前面提到，使用`jsoup`比较合适，因此要引入`jsoup`的jar

```gradle
	compile(
            "org.springframework:spring-webmvc:$spring",
            "org.springframework:spring-context-support:$spring",
            "org.thymeleaf:thymeleaf-spring5:${versions.thymeleaf}",
            "com.alibaba:fastjson:${versions.fastjson}",
            "commons-codec:commons-codec:1.10",
            "commons-io:commons-io:2.5",
            "org.apache.commons:commons-lang3:3.8.1",
            "commons-fileupload:commons-fileupload:1.3.1",
            "org.projectlombok:lombok:1.18.4",
            "org.jsoup:jsoup:1.7.1" //jsoup
    )
```

## 编写JsoupUtils

一开始使用`jsoup`时，遇到一个问题。在富文本框中，把图片转换成`base64`时，传到后台会把`src`给过滤掉，成`<img alt="" width="200"/>`。原因是，在`jsoup`中提供了一个白名单类`org.jsoup.safety.Whitelist`[Whitelist类学习](https://blog.csdn.net/xyw_blog/article/details/9145523)。类中有个方法`addProtocols(java.lang.String,java.lang.String, java.lang.String...)`，可以设置某个标签中的某个特性支持什么协议。看一下白名单，注意`img`

```java
public static Whitelist relaxed() {
    return new Whitelist()
        .addTags(
            "a", "b", "blockquote", "br", "caption", "cite", "code", "col",
            "colgroup", "dd", "div", "dl", "dt", "em", "h1", "h2", "h3", "h4", "h5", "h6",
            "i", "img", "li", "ol", "p", "pre", "q", "small", "span", "strike", "strong",
            "sub", "sup", "table", "tbody", "td", "tfoot", "th", "thead", "tr", "u", "ul"
        )

        .addAttributes("a", "href", "title") // a 标签的合法属性为 href 和 title
        .addAttributes("blockquote", "cite")
        .addAttributes("col", "span", "width")
        .addAttributes("colgroup", "span", "width")
        .addAttributes("img", "align", "alt", "height", "src", "title", "width")
        .addAttributes("ol", "start", "type")
        .addAttributes("q", "cite")
        .addAttributes("table", "summary", "width")
        .addAttributes("td", "abbr", "axis", "colspan", "rowspan", "width")
        .addAttributes("th", "abbr", "axis", "colspan", "rowspan", "scope", "width")
        .addAttributes("ul", "type")

        .addProtocols("a", "href", "ftp", "http", "https", "mailto")
        .addProtocols("blockquote", "cite", "http", "https")
        .addProtocols("cite", "cite", "http", "https")
        .addProtocols("img", "src", "http", "https") //这里img src属性只支持http，和https协议。
        .addProtocols("q", "cite", "http", "https");
}
```

而`base64`是以`data:`开头。所以每次都把`src`给过滤掉了。在白名单中添加：`WHITELIST.addProtocols("img","src","data");//支持img 为base64`就解决了。



```java
package com.peak.utils;

import com.alibaba.fastjson.JSON;
import org.apache.commons.lang3.StringUtils;
import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.safety.Whitelist;

/**
 * 1.使用Jsoup过滤XSS攻击。
 * 2.在Jsoup的Whitelist中内置了很多白名单，这些名单将不会过滤
 */
public class JsoupUtils {

    private static final Whitelist WHITELIST = Whitelist.relaxed();

    static {
        // 富文本编辑时一些样式是使用 style 来进行实现的
        // 比如红色字体 style="color:red;"
        // 所以需要给所有标签添加 style 属性
        WHITELIST.addAttributes(":all", "style");
        WHITELIST.addAttributes(":all", "class");
        WHITELIST.addAttributes(":all", "target");
        WHITELIST.addAttributes(":all", "spellcheck");
        WHITELIST.addProtocols("img","src","data");//支持img 为base64
        System.out.println(JSON.toJSONString(WHITELIST));
    }

    private JsoupUtils() {
    }

    private static final Document.OutputSettings OUTPUT_SETTINGS = new Document.OutputSettings().prettyPrint(false);

    public static String clean(String content) {
        if (StringUtils.isBlank(content)) {
            return "";
        }
        return Jsoup.clean(content, "", WHITELIST, OUTPUT_SETTINGS);
    }

    public static void main(String[] args) {
        String str = "<a href='ss' src='ss'></a><img src='http://www.baidu.com' alt='d' /><script>alert('ss')</script>sss<p style='s' onclick='s'>p标签</p>";
        System.out.println(JsoupUtils.clean(str));
    }
}

```



## 集成HttpServletRequestWrapper

```java
package com.peak.filter;

import com.peak.utils.JsoupUtils;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletRequestWrapper;

public class XSSHttpServletRequestWrapper extends HttpServletRequestWrapper {

    public XSSHttpServletRequestWrapper(HttpServletRequest request) {
        super(request);
    }

    /**
     * 过滤数组参数
     *
     * @param name
     * @return
     */
    @Override
    public String[] getParameterValues(String name) {
        String[] values = super.getParameterValues(name);
        if (null == values)
            return null;
        int count = values.length;
        String[] encodeValues = new String[count];
        for (int i = 0; i < count; i++) {
            encodeValues[i] = JsoupUtils.clean(values[i]);
        }
        return encodeValues;
    }

    /**
     * 过滤参数中的特殊字符
     *
     * @param name
     * @return
     */
    @Override
    public String getParameter(String name) {
        String value = super.getParameter(name);
        if (null == value)
            return null;
        return JsoupUtils.clean(value);
    }

    /**
     * 过滤attribute中的参数
     *
     * @param name
     * @return
     */
    @Override
    public Object getAttribute(String name) {
        Object value = super.getAttribute(name);
        if (null != value && value instanceof String) {
            return JsoupUtils.clean((String) value);
        }
        return value;
    }

    /**
     * 过滤header头部信息
     *
     * @param name
     * @return
     */
    @Override
    public String getHeader(String name) {
        String value = super.getHeader(name);
        if (null == value)
            return value;
        return JsoupUtils.clean(value);
    }
}

```

## 编写Filter

```java
package com.peak.filter;

import jdk.nashorn.internal.objects.annotations.Setter;
import org.apache.commons.lang3.StringUtils;

import javax.servlet.*;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.Arrays;
import java.util.Collections;
import java.util.HashSet;
import java.util.Set;

public class XssFilter implements Filter {
    private Set<String> exUrls;

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        if (StringUtils.isNotBlank(filterConfig.getInitParameter("exUrl"))) {
            this.exUrls = Collections.unmodifiableSet(new HashSet<>(Arrays.asList(filterConfig.getInitParameter("exUrl").split(","))));
        } else {
            this.exUrls = new HashSet<>();
        }

    }

    @Override
    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest) req;
        HttpServletResponse response = (HttpServletResponse) res;
        //获取当前请求路径
        String path = request.getRequestURI().substring(request.getContextPath().length()).replaceAll("[/]+$", "");
        //判断是否在不再过滤范围内 规范:保存或者修改内容时，规范访问地址中包含save或者update
        //这样，只需要添加，或者保存的操作才进行过滤
        if (path.indexOf("save") >= 0 || path.indexOf("update") >= 0) {
            chain.doFilter(new XSSHttpServletRequestWrapper(request), response);
            //chain.doFilter(request, response);
        } else {
            chain.doFilter(request, response);
        }

    }

    @Override
    public void destroy() {

    }
}

```

## 配置Filter

```xml
	<!-- 防止xss攻击 -->
    <filter>
        <filter-name>xssFilter</filter-name>
        <filter-class>com.peak.filter.XssFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>xssFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```

