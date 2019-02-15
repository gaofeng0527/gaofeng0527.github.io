---
title: 在开发中常见的标签以及作用
date: 2017-12-14 17:11:46
tags: [javaweb]
---

在做项目的时候，经常用到一些xml配置文件，一开始我们只是去模仿如何配置，时间久了，就想了解我们使用的这些标签都是什么意思，有什么作用，从现在开始以后就记录下来，一点一点的去了解。

<!-- more -->

# web.xml

---

## `<context-param>`

1.   启动一个WEB项目的时候,容器(如:Tomcat)会去读它的配置文件web.xml.读两个节点: `<listener></listener>` 和` <context-param></context-param>`
2.   紧接着,容器创建一个ServletContext(上下文),这个WEB项目所有部分都将共享这个上下文.
3.   容器将`<context-param></context-param>`转化为键值对,并交给ServletContext.
4.   容器创建`<listener></listener>`中的类实例,即创建监听.
5.   在监听中会有contextInitialized(ServletContextEvent args)初始化方法,在这个方法中获得ServletContext = ServletContextEvent.getServletContext();

[参考Blog](http://blog.csdn.net/sxbjffsg163/article/details/9955479)



# spring.xml

---

## `<context:component-scan>`

通常情况下我们在创建spring项目的时候在xml配置文件中都会配置这个标签，配置完这个标签后，spring就会去自动扫描base-package对应的路径或者该路径的子包下面的java文件，如果扫描到文件中带有@Service,@Component,@Repository,@Controller等这些注解的类，则把这些类注册为bean

**注：**如果配置了`<context:component-scan>`那么`<context:annotation-config>`标签就可以不用在xml中再配置了，因为前者包含了后者。

[详情参考](http://blog.csdn.net/liuxingsiye/article/details/52171508)

