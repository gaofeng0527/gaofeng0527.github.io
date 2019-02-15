---
title: 搭建一个spring-springmvc-mybitis项目
date: 2017-12-15 11:23:01
tags: [javaweb,springmvc]
---

# 背景

平时做项目都是在前人搭建好的项目上进行开发，虽然对框架有了一定的了解，能完成开发任务，可是自己始终没有完整的搭建过ssm框架，并且对一个**web**项目中所应用的技术没有一个好的认知和总结。因此自己总结了一下，以备以后使用。并希望在以后的学习中，不断完善该文档。

<!-- more -->

# 技术介绍

项目管理：maven

集成框架：使用轻量级的spring框架

控制层框架：使用springmvc

视图模板：thymeleaf

数据层：使用mybitis3

日志框架：logback

数据库连接池：dbcp2

数据库：mysql

# 添加spring，springmvc应用

## 添加依赖

```xml
<properties>
	<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
	<project.build.locales>zh_CN</project.build.locales>
	<project.build.jdk>1.7</project.build.jdk>
	<plugin.maven-compiler>3.1</plugin.maven-compiler>
	<spring.version>5.0.0.RELEASE</spring.version>
</properties>
<!-- spring -->
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-core</artifactId>
	<version>${spring.version}</version>
</dependency>
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-oxm</artifactId>
	<version>${spring.version}</version>
</dependency>
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-tx</artifactId>
	<version>${spring.version}</version>
</dependency>
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-jdbc</artifactId>
	<version>${spring.version}</version>
</dependency>
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-aop</artifactId>
	<version>${spring.version}</version>
</dependency>
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-context-support</artifactId>
	<version>${spring.version}</version>
</dependency>
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-test</artifactId>
	<version>${spring.version}</version>
</dependency>
<!-- springmvc -->
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-webmvc</artifactId>
	<version>${spring.version}</version>
</dependency>
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-web</artifactId>
	<version>${spring.version}</version>
</dependency>
<!-- 可以使用j2ee提供的几个注解比如@resource -->
<dependency>
	<groupId>javax.annotation</groupId>
	<artifactId>javax.annotation-api</artifactId>
	<version>1.2</version>
</dependency>
<!-- 处理json数据 -->
<dependency>
	<groupId>com.alibaba</groupId>
	<artifactId>fastjson</artifactId>
	<version>1.2.41</version>
</dependency>
```



## 添加spring配置

`applicationContext.xml` spring的主配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd">
	<!-- spring可以自动去扫描base-pack下面或者子包下面的java文件，
	如果扫描到有@Component @Controller@Service等这些注解的类，则把这些类注册为bean -->
	<context:component-scan base-package="com.gaofeng"></context:component-scan>
</beans>

```

`springmvc-servlet.xml` springmvc的配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
	xmlns:mvc="http://www.springframework.org/schema/mvc" xmlns:tx="http://www.springframework.org/schema/tx"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd
		http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.3.xsd
		http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.3.xsd">
	<!-- 配置文件 -->
	<context:property-placeholder location="classpath:application.properties" />

	<!-- 扫描控制器 -->
	<context:component-scan base-package="com.gaofeng.controller"></context:component-scan>
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
			<bean
				class="com.alibaba.fastjson.support.spring.FastJsonHttpMessageConverter4">
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
	<!-- 静态资源的访问，如 js, css, jpg, png -->
	<!-- 如 HTML 里访问 /static/js/foo.js, 则实际访问的是 /WEB-INF/static/js/foo.js -->
	<mvc:resources mapping="/static/js/**" location="/WEB-INF/static/js/"
		cache-period="31556926" />
	<mvc:resources mapping="/static/css/**" location="/WEB-INF/static/css/"
		cache-period="31556926" />
	<mvc:resources mapping="/static/img/**" location="/WEB-INF/static/img/"
		cache-period="31556926" />
	<mvc:resources mapping="/static/html/**" location="/WEB-INF/static/html/"
		cache-period="31556926" />
</beans>

```



## 修改web.xml配置加载spring配置文件

```xml
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
	<display-name>Archetype Created Web Application</display-name>
	<!-- POST 请求的编码 -->
	<filter>
		<filter-name>characterEncodingFilter</filter-name>
		<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
		<init-param>
			<param-name>encoding</param-name>
			<param-value>UTF-8</param-value>
		</init-param>
		<init-param>
			<param-name>forceEncoding</param-name>
			<param-value>true</param-value>
		</init-param>
	</filter>
	<filter-mapping>
		<filter-name>characterEncodingFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>

	<!-- 启动一个WEB项目的时候,容器(如:Tomcat)会去读它的配置文件web.xml.读两个节点: <listener></listener> 
		和 <context-param></context-param> 紧接着,容器创建一个ServletContext(上下文),这个WEB项目所有部分都将共享这个上下文. 
		容器将<context-param></context-param>转化为键值对,并交给ServletContext. 容器创建<listener></listener>中的类实例,即创建监听. 
		这里是把spring主配置文件添加到servlet上下文中 -->
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath:applicationContext.xml</param-value>
	</context-param>

	<!-- Bootstraps the root web application context before servlet initialization -->
	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>

	<!-- 配置springmvc核心selvlet-->
	<servlet>
		<servlet-name>springDispatcherServlet</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>classpath:springmvc-servlet.xml</param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>
	</servlet>

	<!-- Map all requests to the DispatcherServlet for handling -->
	<servlet-mapping>
		<servlet-name>springDispatcherServlet</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>
</web-app>

```

通过上面的配置，我们已经添加了spring和springmvc框架，来验证一下。

## 编写各层的bean

项目使用多层的设计风格分为控制层，业务逻辑层，数据访问层，视图层；使用接口编程，降低各层的耦合性。

Person.java

```java
package com.gaofeng.pojo;

public class Person {
	
	private Long id;
	private String uname;
	private String email;
	//省略setter和getter

}

```

PersonServic.java 接口

```java
package com.gaofeng.service;

import com.gaofeng.pojo.Person;

public interface PersonService {

	public void save(Person person);

}
```

PersonServiceImpl.java实现类

```java
package com.gaofeng.service.impl;

import org.springframework.stereotype.Service;

import com.gaofeng.pojo.Person;
import com.gaofeng.service.PersonService;

@Service(value="pesonService")
public class PersonServiceImpl implements PersonService {
	
	public void save(Person person) {
		System.out.println("save peson :" + person.toString());
	}
}

```

IndexController.java 控制层

```java
package com.gaofeng.controller;

import javax.annotation.Resource;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.ResponseBody;

import com.gaofeng.pojo.Person;
import com.gaofeng.service.PersonService;

@Controller
public class IndexController {

	@Resource(name = "pesonService")
	private PersonService pservice;

	@GetMapping("/index")
	public String index(Model model) {
		model.addAttribute("message", "欢迎来到spring满汉全席");
		return "index";
	}

	@GetMapping("/test/add")
	public String addPerson(Model model) {
		Person p = new Person();
		p.setId(1L);
		p.setUname("张高峰");
		p.setEmail("gaofeng0527@163.com");
		pservice.save(p);
		model.addAttribute("message", "您已经成功添加一个用户");
		return "index";
	}

}

```

>    `@Resource` 注解会先根据bean的名字去IOC容器中查找bean，如果没有的话才会使用类型匹配。找到后把spring ioc容器创建的bean交给IndexController使用。

## 测试

>    连接：http://localhost/gaofeng/index
>
>    页面显示：欢迎来到spring满汉全系
>
>    连接：http://localhost/gaofeng/test/add
>
>    控制台输出：save peson :Person [id=1, uname=张高峰, email=gaofeng0527@163.com]
>
>    页面显示：您已经成功添加一个用户

## 总结

spring，springmvc添加成功，可以实现依赖注入和控制反转，这也是使用spring的最主要的作用。

# 添加mybitis

## 添加依赖

因为需要连接数据，所以要添加数据库驱动包；mybitis的包自不用说；现在大部分项目都在使用数据库连接池，因此需要加入数据库连接池的包，这里使用dbcp2。

>    在spring中，常使用数据库连接池来完成对数据库的连接配置，类似于线程池的定义，数据库连接池就是维护有一定数量数据库连接的一个缓冲池，一方面，能够即取即用，免去初始化的时间，另一方面，用完的数据连接会归还到连接池中，这样就免去了不必要的连接创建、销毁工作，提升了性能。当然，使用连接池，有一下几点是连接池配置所考虑到的，也属于配置连接池的优点，而这些也会我们后面的实例配置中体现： 
>    1、 如果没有任何一个用户使用连接，那么那么应该维持一定数量的连接，等待用户使用。 
>    2、 如果连接已经满了，则必须打开新的连接，供更多用户使用。 
>    3、 如果一个服务器就只能有100个连接，那么如果有第101个人过来呢？应该等待其他用户释放连接 
>    4、 如果一个用户等待时间太长了，则应该告诉用户，操作是失败的。
>
>    在spring中，常用的连接池有：jdbc,dbcp,c3p0,JNDI4种，他们有不同的优缺点和适用场景。其中，spring框架推荐使用dbcp，hibernate框架推荐使用c3p0。经测试发现，c3p0与dbcp相比较，c3p0能够更好的支持高并发，但是在稳定性方面略逊于dpcp。 

```xml
<!-- database -->
<dependency>
	<groupId>org.apache.commons</groupId>
	<artifactId>commons-dbcp2</artifactId>
	<version>2.1.1</version>
</dependency>
<!-- DataBase数据库连接 mysql包 -->
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
	<version>5.1.38</version>
</dependency>
<!-- mybitis -->
<dependency>
	<groupId>org.mybatis</groupId>
	<artifactId>mybatis</artifactId>
	<version>3.4.1</version>
</dependency>
<dependency>
	<groupId>org.mybatis</groupId>
	<artifactId>mybatis-spring</artifactId>
	<version>1.3.0</version>
</dependency>
```

## 添加配置

为了方便后期维护，把配置分成三部分：

1.   application.properties 主要用来存储一些系统配置，比如数据库连接信息
2.   spring-data.xml 主要用来配置数据库连接
3.   applicationContext.xml spring主配置，把spring-data.xml引入进来

`spring-data.xml` 数据库连接和mybitis添加

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.3.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd">

	<!-- 配置数据源 -->
	<!-- destroy-method="close" 设置为close使Spring容器关闭同时数据源能够正常关闭，以免造成连接泄露 -->
	<bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource"
		destroy-method="close">
		<property name="driverClassName" value="com.mysql.jdbc.Driver" />
		<property name="url"
			value="jdbc:mysql://localhost:3306/gaofeng?useUnicode=true&amp;characterEncoding=UTF-8" />
		<property name="username" value="root" />
		<property name="password" value="" />
		<!-- 连接池启动初始连接数 -->
		<property name="initialSize" value="5" />
		<!-- 连接池最大连接数 -->
		<property name="maxTotal" value="100" />
		<!-- 最大空闲值，当经过一个高峰之后，连接池会慢慢的把不用的连接释放，一直减少到maxIdle值为止 -->
		<property name="maxIdle" value="50" />
		<!-- 最小空闲值,当空闲的连接数少于minIdle的值时，连接池会预先申请一部分连接，以防洪峰来时，来不及申请 -->
		<property name="minIdle" value="5" />
		<!-- 即启用poolPreparedStatements后，PreparedStatements 和CallableStatements 
			都会被缓存起来复用，即相同逻辑的SQL可以复用一个游标，这样可以减少创建游标的数量。 -->
		<property name="poolPreparedStatements" value="true" />
		<property name="maxOpenPreparedStatements" value="10" />

		<!-- 给出一条简单的sql语句进行验证 -->
		<property name="validationQuery" value="select NOW()" />
		<!-- 在取出连接时进行有效验证, 实现如服务器重启后自动重连 -->
		<property name="testOnBorrow" value="false" />
		<property name="testWhileIdle" value="true" />
		<property name="logAbandoned" value="true" />
		<property name="removeAbandonedTimeout" value="120" />
		<!-- 运行判断连接超时任务的时间间隔，单位为毫秒，默认为-1，即不执行任务 -->
		<property name="timeBetweenEvictionRunsMillis" value="3600000" />
		<!-- 连接的超时时间，默认为半小时 -->
		<property name="minEvictableIdleTimeMillis" value="3600000" />
	</bean>
	<!-- 2. SQL session factory -->
	<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
		<property name="dataSource" ref="dataSource" />
		<property name="mapperLocations" value="classpath:com/gaofeng/mapper/**/*.xml" /> 
	</bean>
	<!-- 3. Instantiate Mapper -->
	<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
		<property name="basePackage" value="com.gaofeng.mapper" />
	</bean>
</beans>
```

>    `org.mybatis.spring.mapper.MapperScannerConfigurer` 类会为basePackage包下的所有类作为mybitis的mapper注册到ioc容器中，并创建bean。方便后期自动注入。

修改`applicationContext.xml`，导入`spring-data.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd">

	<import resource="classpath:spring-data.xml"/>
	<!-- spring可以自动去扫描base-pack下面或者子包下面的java文件，
	如果扫描到有@Component @Controller@Service等这些注解的类，则把这些类注册为bean -->
	<context:component-scan base-package="com.gaofeng"></context:component-scan>
</beans>
```

## 编写mapper接口和配置文件

PersonMapper接口

```java
package com.gaofeng.mapper;

import com.gaofeng.pojo.Person;

public interface PersonMapper {
	
	public Person findPersonById(Long id);

}
```

PersonMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- 1.命名空间要和接口全名称一致 
     2.sql语句映射配置的Id要和接口中的方法一致 
     3.查询中如果类属性和数据库表中的字段名称不一致时，可以采用 
	select user_name as userName 的形式 
     4.返回类型最好使用类全名，也可以采用别名的形式。 -->
<mapper namespace="com.gaofeng.mapper.PersonMapper">
	<select id="findPersonById" resultType="com.gaofeng.pojo.Person">
		select * from person where id = #{id}
	</select>
</mapper>
```

## 数据准备工作

创建数据库，创建表，添加测试数据，在这里就不再讲了。

## 修改之前的service.impl和controller

修改PersonService.java，添加findPersonById方法

```java
package com.gaofeng.service;

import com.gaofeng.pojo.Person;

public interface PersonService {

	public void save(Person person);

	public Person findPersonById(Long id);

}

```

修改PersonServiceImpl.java

```java
package com.gaofeng.service.impl;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import com.gaofeng.mapper.PersonMapper;
import com.gaofeng.pojo.Person;
import com.gaofeng.service.PersonService;

@Service(value="pesonService")
public class PersonServiceImpl implements PersonService {
	
	/**
	 * 这个地方因为之前在spring-data.xml配置文件中使用
	 * MapperScannerConfigurer 扫描 mapper包，并创建了bean
	 * 因此这里可以自动注入
	 */
	@Autowired
	private PersonMapper pmapper;

	public void save(Person person) {
		System.out.println("save peson :" + person.toString());
	}
	
	public Person findPersonById(Long id) {
		return pmapper.findPersonById(id);
	}

}
```

IndexController.java，添加`/dome/{id}`处理方法

```java
package com.gaofeng.controller;

import javax.annotation.Resource;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.ResponseBody;

import com.gaofeng.pojo.Person;
import com.gaofeng.service.PersonService;

@Controller
public class IndexController {

	@Resource(name = "pesonService")
	private PersonService pservice;

	@GetMapping("/index")
	public String index(Model model) {
		model.addAttribute("message", "欢迎来到spring满汉全席");
		return "index";
	}

	@GetMapping("/test/add")
	public String addPerson(Model model) {
		Person p = new Person();
		p.setId(1L);
		p.setUname("张高峰");
		p.setEmail("gaofeng0527@163.com");
		pservice.save(p);
		model.addAttribute("message", "您已经成功添加一个用户");
		return "index";
	}

	@GetMapping("/dome/{id}")
	@ResponseBody
	public Person findPerson(@PathVariable(value = "id") Long id) {
		return pservice.findPersonById(id);
	}
}

```



## 测试

>    访问连接：http://localhost/gaofeng/dome/1
>
>    页面显示：{"email":"gaofeng0527@163.com","id":1,"uname":"张高峰"}
>
>    控制台：输出sql语句

自此，三大框架集成完毕。

# 集成logback日志框架

之前已经写过这个了，把连接贴下面，以供参考吧

[集成logback日志框架](https://gaofeng0527.github.io/logback/)

# 使用thymeleaf作为前端模版

这个之前也写过了

[springmvc集成thymeleaf](https://gaofeng0527.github.io/thymeleaf/)



# 总结

web开发，这些都是基础，在开发过程中还会遇到很多问题，希望以后没遇到一个问题，自己都能记录下来。每天让自己进步一点。哎，有些后悔之前几年蹉跎了岁月。