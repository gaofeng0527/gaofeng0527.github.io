---
title: thymeleaf处理URL连接
date: 2017-12-04 19:07:15
tags: [thymeleaf]
---

URL是Web应用程序模板中的一等公民，而Thymeleaf标准方言具有特殊的语法，@语法：@ {...}

# th:href

## @语法:@{...}

```html
<!-- 绝对网址-->
<a href="details.html" th:href="@{http://localhost:8080/gtvg/order/details(orderId=${o.id})}">view</a>
<!-- 服务器相对路径 -->
<a href="details.html" th:href="@{/order/details(orderId=${o.id})}">view</a>
<!-- 路径中也可以使用变量模板-->
<a href="details.html" th:href="@{/order/{orderId}/details(orderId=${o.id})}">view</a>
```

注意：

>    1.   th:href是一个修饰符属性：一旦处理，它将计算要使用的连接URL，并将该值设置为<a>表情的href属性的值。
>    2.   URL中的参数也可以用表达式，例如http://localhost:8080/gtvg/order/details(orderId=${o.id})，所需的URL参数编码，thymeleaf也会自动执行。
>    3.   如果需要多个参数，可以用逗号隔开@ {/order/process(execId=${execId},execType='FAST')} 
>    4.   URL路径中也允许使用变量模板：@{/order/{orderId}/details(orderId=${orderId})}
>    5.   以/开头的请求，会自动匹配上下文前缀
>    6.   th:href属性允许和静态的href属性共存，这样我们可以在浏览器直接打开原型查看效果

与消息语法（#{...}）的情况一样，URL基数也可以是计算另一个表达式的结果：

```html
<a th:href="@{${url}(orderId=${o.id})}">view</a>
<a th:href="@{'/details/'+${user.login}(orderId=${o.id})}">view</a>
```

