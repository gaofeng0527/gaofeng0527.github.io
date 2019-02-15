---
title: eclipse 创建maven项目，并添加spring依赖
date: 2016-11-28 11:56:21
tags: [spring,maven]
---

## Maven 下载安装

>    1.先去官网下载需要的maven版本 :http://maven.apache.org/
>
>    下载完成后，解压到想要安装的目录即可，免安装的。
>
>    2.配置Maven环境变量，和jdk环境变量配置大同小异，可参考：http://jingyan.baidu.com/article/cb5d61050b8ee7005d2fe04e.html

<!--more-->

## Eclipse添加Maven支持

>    1.打开eclipse的Help菜单Help->Eclipse Marketplace搜索关键字maven到插件Maven Integration for Eclipse 并点击安装即可
>
>    2.安装完成后，重启eclipse；为了使得Eclipse中安装的Maven插件，同windows中安装的那个相同，需要让eclipse中的maven重新定位一下，点击Window -> Preference -> Maven -> Installation -> Add进行设置

Maven还有很多内容，这里不是以Maven为主，所以要了解更多的Maven知识请参考：http://www.yiibai.com/maven/

# eclipse篇

## 使用maven插件在eclipse中创建Maven项目

1.File-->New-->other-->Maven-->Maven Project-->next 创建Maven项目，如图：

![](http://ohhigdkar.bkt.clouddn.com/1.png)



![](http://ohhigdkar.bkt.clouddn.com/2.png)

![](http://ohhigdkar.bkt.clouddn.com/3.png)

![](http://ohhigdkar.bkt.clouddn.com/4.png)

成功创建带有pom.xml文件的项目工程

![](http://ohhigdkar.bkt.clouddn.com/5.png)

# Maven & Spring篇

## pom.xml详解

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.spring</groupId><!-- 项目或组织的唯一标志，并且配置时生成的路径也是依据这个配置的，例如 com.spring  生成的目录com/spring -->
  <artifactId>mvn</artifactId><!-- 项目的通用名 -->
  <packaging>war</packaging><!-- 项目打包方式，常用方式有war,jar等 -->
  <version>0.0.1-SNAPSHOT</version><!-- 版本 -->
  <name>mvn Maven Webapp</name><!-- 描述信息，可选 -->
  <url>http://maven.apache.org</url><!-- 一般都会写开发团队或者公司的连接，可选 -->
  
  <!-- 设置上下文属性 -->
  <properties>
  	<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding><!-- 统一项目编码 -->
  	<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding><!-- 输出统一编码 -->
  	<java.version>1.8</java.version><!-- jdk版本 -->
  </properties>
  
  <!-- 依赖关系-->
  <dependencies>
    <dependency>
      <groupId>junit</groupId><!-- 添加junit依赖，依赖的坐标可以去maven中央仓库查询 -->
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
       <!-- Scope表示这个library存在的时期，有compile，runtime，test和provided，provided表示在运行时这个app所在的device会给他提供,test表示测试的时候 -->  
      <scope>test</scope>
    </dependency>
  </dependencies>
  
   <!-- Build 节点配置的是项目编译的时候需要用到的一些东西,一个build节点下面有一个plugins子节点，在下面有多个plugin子节点，每个plugin子节点有groupId，artifactId，version等子节点，含义跟上面说到的基本一样。此外，plugin节点一般还有configuration节点，表示对这个plugin进一步的配置-->  
  <build>
    <finalName>mvn</finalName>
  </build>
</project>

```



## 在项目中添加spring依赖

1.确定要使用的spring版本，在*properties* 节点中添加下面语句

```xml
<org.springframework.version>4.1.6.RELEASE</org.springframework.version><!-- 方便控制spring版本统一 -->
```

2.添加spring-context依赖，可以先去maven中央仓库http://http://mvnrepository.com/中查找坐标，如图：

![](http://ohhigdkar.bkt.clouddn.com/springse.png)



获取spring context的坐标

![](http://ohhigdkar.bkt.clouddn.com/springcontext.png)

把下面的依赖粘贴进pom.xml配置文件中的*dependencies*节点下。因为我们上面配置过spring的版本，所以这里的version可已修改如下：

```xml
<!-- spring context 依赖 -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-context</artifactId>
			<version>${org.springframework.version}</version>
			<scope>runtime</scope><!-- 项目运行时加载依赖 -->
		</dependency>
```

这里有一个地方需要注意一下，*javax.servlet* 有的开发环境中是没有这个包的，也需要添加依赖，但是在正事运行环境是自带这个包的，也就是说打包部署的时候要去掉这个依赖，因此在开发环境配置该依赖的时候，要设置*scope* 属性，如：

```xml
        <dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>servlet-api</artifactId>
			<version>2.4</version>
			<scope>provided</scope>
		</dependency>  
```

>    目前<scope>可以使用5个值：
>
>       1.compile，缺省值，适用于所有阶段，会随着项目一起发布。 
>
>    2.   provided，类似compile，期望JDK、容器或使用者会提供这个依赖。如servlet.jar。
>
>    3.   runtime，只在运行时使用，如JDBC驱动，适用运行和测试阶段。 
>
>    4.   test，只在测试时使用，用于编译和运行测试代码。不会随项目发布。
>
>    5.   system，类似provided，需要显式提供包含依赖的jar，Maven不会在Repository中查找它。
>
>         ​



3.其他依赖如上步骤，添加完依赖后的pom.xml如下：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.spring</groupId><!-- 项目或组织的唯一标志，并且配置时生成的路径也是依据这个配置的，例如 com.spring 
		生成的目录com/spring -->
	<artifactId>mvn</artifactId><!-- 项目的通用名 -->
	<packaging>war</packaging><!-- 项目打包方式，常用方式有war,jar等 -->
	<version>0.0.1-SNAPSHOT</version><!-- 版本 -->
	<name>mvn Maven Webapp</name><!-- 描述信息，可选 -->
	<url>http://maven.apache.org</url><!-- 一般都会写开发团队或者公司的连接，可选 -->

	<!-- 设置上下文属性 -->
	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding><!-- 
			统一项目编码 -->
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding><!-- 
			输出统一编码 -->
		<java.version>1.8</java.version><!-- jdk版本 -->
		<org.springframework.version>4.1.6.RELEASE</org.springframework.version><!-- 
			方便控制spring版本统一 -->
	</properties>

	<!-- 依赖关系 -->
	<dependencies>
		<dependency>
			<groupId>junit</groupId><!-- 添加junit依赖，依赖的坐标可以去maven中央仓库查询 -->
			<artifactId>junit</artifactId>
			<version>3.8.1</version>
			<!-- Scope表示这个library存在的时期，有compile，runtime，test和provided，provided表示在运行时这个app所在的device会给他提供,test表示测试的时候 -->
			<scope>test</scope>
		</dependency>
		<!-- spring context 依赖 上例定义的对spring-context的依赖，spring-context实现了Spring注入容器并且依赖：spring-core,spring-expression,spring-aop以及spring-beans。这些依赖包使容器可以支持Spring的一些核心技术：Spring核心组件,Spring 
			EL表达式 (SpEL), 面向切面编程,JavaBean机制。 -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-context</artifactId>
			<version>${org.springframework.version}</version>
			<scope>runtime</scope><!-- 项目运行时加载依赖 -->
		</dependency>

		<!-- spring 持久化依赖 -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-orm</artifactId>
			<version>${org.springframework.version}</version>
		</dependency>
		
		<!--spring mvc 要增加Spring Web和Servlet支持，需要在上面已配置的pom文件中额外增加两个依赖 -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-web</artifactId>
			<version>${org.springframework.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-webmvc</artifactId>
			<version>${org.springframework.version}</version>
		</dependency>

	</dependencies>
	<!-- Build 节点配置的是项目编译的时候需要用到的一些东西,一个build节点下面有一个plugins子节点，在下面有多个plugin子节点，每个plugin子节点有groupId，artifactId，version等子节点，含义跟上面说到的基本一样。此外，plugin节点一般还有configuration节点，表示对这个plugin进一步的配置 -->
	<build>
		<finalName>mvn</finalName>
	</build>
</project>
```



## 编写spring mvc例子

### 配置DispatcherServlet

```java
package config;

import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer;
/**
 * 传统的方式，web应用添加spring mvc框架需要在web.xml配置文件中添加DispatcherServlet映射
 * 借助于Servlet3规范和spring 3.1的功能增加，我们可以在java类中进行DispatcherServlet的映射了
 * AbstractAnnotationConfigDispatcherServletInitializer 类剖析
 * 在Servlet3.0环境中，容器会在类路径中查找实现javax.servlet.ServletContainerInitializer接口类，如果能发现的话，就用来配置Servlet容器。
 * spring提供了这个接口的实现，名为SpringServletContainerInitializer，这个类反过来又会查找实现WebApplicationInitializer的类并将配置的任务
 * 交给他们来完成。Spring 3.2进入了一个便利的WebApplicationInitializer基础实现，也就是AbstractAnnotationConfigDispatcherServletInitializer。
 * 因此，当我们的类实现了这个接口，同时也就实现了WebApplicationInitializer，所以当部署到servlet3.0容器中的时候，容器会自动发现它，并用它来配置servlet上下文
 * @author Administrator
 *
 */
public class SpringWebAppInitialazer extends AbstractAnnotationConfigDispatcherServletInitializer {

	@Override
	protected Class<?>[] getRootConfigClasses() {
		return new Class<?>[]{RootConfig.class};
	}

	/**
	 * 指定一个配置类
	 */
	@Override
	protected Class<?>[] getServletConfigClasses() {
		return new Class<?>[]{WebConfig.class};
	}

	/**
	 * 将一个或多个路径映射到DispatcherServlet上
	 * 也就是所有的请求都先到DispactcherServlet前端控制器中（如果不明白，先去了解一下spring mvc运行原理）
	 */
	@Override
	protected String[] getServletMappings() {
		return new String[]{"/"};
	}

}

```

### 创建java配置类

```java
package config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.ViewResolver;
import org.springframework.web.servlet.config.annotation.DefaultServletHandlerConfigurer;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
import org.springframework.web.servlet.view.InternalResourceViewResolver;

import controller.CController;

/**
 * spring 配置类，DispactcherServlet会使用该类配置的Bean
 * ComponentScan 注解会自动扫描某个包，这里用了basePackageClasses会已配置的某个类为基础扫描该类所在包，或该包的子包
 * 使用该方式方便后期重构
 * @author Administrator
 *
 */
@EnableWebMvc
@Configuration
@ComponentScan(basePackageClasses={CController.class})
public class WebConfig extends WebMvcConfigurerAdapter{

	/**
	 * 配置jsp视图解析器
	 * @return
	 */
	@Bean
	public ViewResolver viewResolver(){
		InternalResourceViewResolver resolver = new InternalResourceViewResolver();
		resolver.setPrefix("/WEB-INF/views/");
		resolver.setSuffix(".jsp");
		resolver.setExposeContextBeansAsAttributes(true);
		return resolver;
	}

	/**
	 * 配置静态资源的处理
	 * DispatcherServlet将对静态资源的请求转发到Servlet容器默认的Servlet上，而不是DispatcherServlet本身处理
	 */
	@Override
	public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
		configurer.enable();
	}
	
	
}

```

### 创建RootConfig.java

```java
package config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

import controller.CController;

/**
 * 该类在本实例中没有用到，暂不解释
 * @author Administrator
 *
 */
@Configuration
@ComponentScan(basePackageClasses={CController.class})
public class RootConfig {

}

```

### 编写controller

```java
package controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

@Controller
public class IndexController {

	@RequestMapping(value="/index",method=RequestMethod.GET)
	public String index() {
		return "index";
	}
}

```

### 编写jsp

```html
<html>
<body>
<h2>Hello World!</h2>
</body>
</html>

```

### 启动tomcat

>    http://localhost:8077/springHello/index 测试

![](http://ohhigdkar.bkt.clouddn.com/hell.png)

### 注意

>    1.该springmvc实例需要在servlet 3.0容器，tomcat7.0或以上版本运行

