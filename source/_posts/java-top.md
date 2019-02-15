---
title: 日常积累
date: 2019-02-15 11:52:32
tags: java
---

# 比较Long类型 20190215

平时比较数字类型时习惯直接用`==`来比较，有一次发现两个Long类型的数字直接用`==`比较没有达到预期效果，仔细一想Long类型是long的封装类型，`==`比较是比较的引用地址，因此值是一样的，却返回false。

```java
Long l1 = new Long(10);
Long l2 = new Long(10);
System.out.println(l1 == l2);//false
System.out.println(l1.longValue() == l2.longValue());//true
```

上面代码也可以看出：long类型是java中的基本数据类型，可以用`==`直接比较

<!-- more -->

# cookie的读取 20190215

之前没有添加这一句`cookie.setPath("/");`导致在读取cookie的时候失败，是因为cookie默认的作用域是在同一路径下。`/write/cookie` 和 `/read/cookie` 不属于同一路径。

```java
@RequestMapping("/write/cookie")
    @ResponseBody
    public String writeCookie(HttpServletResponse response) {
        Cookie cookie = new Cookie("username", "Don't tell me");
        cookie.setMaxAge(10000);
        cookie.setPath("/");
        response.addCookie(cookie);
        return "write cookie username";
    }
```

```java
/**
     * 在添加：cookie.setPath("/");这一行代码之前，读取不到cookie，因为cookie默认的作用域是在同一路径下有效
     * 添加这一行代码后，是在同一域名下有效
     * @param cookie
     * @return
     */
    @RequestMapping("/read/cookie")
    @ResponseBody
    public String readCookie(@CookieValue(value = "username",defaultValue = "") String cookie) {
        return "cookie:username ==> "+cookie;
    }
```

# GitHub保存笔记源文件20190215

1.   在`hexo`下创建笔记 

     >    hexo new "fileName"
     >
     >    hexo generate
     >
     >    hexo deploy 

2.   在Blog目录下执行git命令把修改添加到Git仓库中

     >    git add <file>
     >
     >    git commit -m "message"
     >
     >    git push -u origin master

这样，换到任何环境，都不会丢失自己之前写过的笔记了（除非github不再免费存储了）