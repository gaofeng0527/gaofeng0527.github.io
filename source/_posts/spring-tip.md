---
title: spring tip
date: 2018-06-29 09:45:17
tags: spring
---

# 导入jar

```xml
dependencies {

    compile(
            "org.springframework:spring-webmvc:${versions.spring}",
            "org.springframework:spring-context-support:${versions.spring}",
            "org.springframework:spring-jdbc:${versions.spring}"
    )
    testCompile group: 'junit', name: 'junit', version: '4.11'
    testCompile group: 'junit', name: 'junit', version: '4.12'
    testCompile("org.springframework:spring-test:${versions.spring}")
}

```

<!-- more -->

>    1.   org.springframework:spring-aop:5.0.2.RELEASE
>    2.   spring-beans:
>    3.   spring-context
>    4.   spring-context-support
>    5.   spring-core
>    6.   spring-expression
>    7.   spring-jcl
>    8.   spring-jdbc
>    9.   spring-test
>    10.   spring-tx
>    11.   spring-web
>    12.   spring-webmvc
>
>

# 创建application.xml

spring的主配置文件，现在大部分都是用注解开发了，因此该配置文件并不主要用来配置bean的生成，可以打开注解扫描，配置系统其它主要配置项，导入其它模块配置文件等。总之作为一个主导整个程序的上下文。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:util="http://www.springframework.org/schema/util"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">



    <context:property-placeholder location="classpath:config/config.properties"></context:property-placeholder>

    <bean id="domeService" class="example.spring.service.impl.DomeServiceImpl">
    </bean>
    <!-- 1.属性注入 -->
    <bean id="dome" class="example.spring.beans.Dome">
        <property name="name" value="${uname}"></property>
        <property name="message" value="${pwd}"></property>
    </bean>
    <!-- 1.构造方法注入 -->
    <bean id="dome1" class="example.spring.beans.Dome">
        <constructor-arg name="name" value="好好"></constructor-arg>
        <constructor-arg name="message" value="好好"></constructor-arg>
    </bean>

    <bean id="dome3" class="example.spring.beans.Dome">
        <constructor-arg value="吃饭" index="1"></constructor-arg>
        <constructor-arg value="好好" index="0"></constructor-arg>
    </bean>

    <!-- 当字面值有特殊字符的时候，需要使用<![CDATA[]]>，但是需要使用value子标签 -->
    <bean id="dome4" class="example.spring.beans.Dome">
        <property name="name">
            <value><![CDATA[<张高峰>]]></value>
        </property>
        <property name="message" value="测试"></property>
    </bean>
    <bean id="dome5" class="example.spring.beans.Dome">
        <property name="name" value="张高峰"></property>
        <property name="message" value="测试"></property>
        <property name="adds">
            <list>
                <value type="java.lang.String">add1</value>
                <value type="java.lang.String">add2</value>
                <value type="java.lang.String">add3</value>
            </list>
        </property>
    </bean>
</beans>
```

## 属性注入

```xml
	 <!-- 1.属性注入 -->
    <bean id="dome" class="example.spring.beans.Dome">
        <property name="name" value="gaofeng"></property>
        <property name="message" value="say hello"></property>
    </bean>
    <!-- 当字面值有特殊字符的时候，需要使用<![CDATA[]]>，但是需要使用value子标签 -->
    <bean id="dome4" class="example.spring.beans.Dome">
        <property name="name">
            <value><![CDATA[<张高峰>]]></value>
        </property>
        <property name="message" value="测试"></property>
    </bean>
	<!-- 当有list属性或者数组属性时，可以使用list标签注入 -->
  	<bean id="dome5" class="example.spring.beans.Dome">
          <property name="name" value="张高峰"></property>
          <property name="message" value="测试"></property>
          <property name="adds">
              <list>
                  <value type="java.lang.String">add1</value>
                  <value type="java.lang.String">add2</value>
                  <value type="java.lang.String">add3</value>
              </list>
          </property>
      </bean>
```

##  构造器注入

```xml
 	<!-- 1.构造方法注入 -->
    <bean id="dome1" class="example.spring.beans.Dome">
        <constructor-arg name="name" value="好好"></constructor-arg>
        <constructor-arg name="message" value="好好"></constructor-arg>
    </bean>

    <bean id="dome3" class="example.spring.beans.Dome">
        <constructor-arg value="吃饭" index="1"></constructor-arg>
        <constructor-arg value="好好" index="0"></constructor-arg>
    </bean>

```

## 引入外部属性文件

```xml
	<context:property-placeholder location="classpath:config/config.properties">				</context:property-placeholder>
	<bean id="dome" class="example.spring.beans.Dome">
        <property name="name" value="${uname}"></property>
        <property name="message" value="${pwd}"></property>
    </bean>	

```

## ref注入

```xml
	<bean id="domeService" class="example.spring.service.impl.DomeServiceImpl">
        <property name="dome" ref="dome"></property>
    </bean>
    <!-- 1.属性注入 -->
    <bean id="dome" class="example.spring.beans.Dome">
        <property name="name" value="${uname}"></property>
        <property name="message" value="${pwd}"></property>
    </bean>
```

## p标签

引入p标签命名空间

```xml
	<bean id="dome" class="example.spring.beans.Dome" p:name="jom" p:message="a boy"></bean>
```

## 作用域创建实例

有四个作用域的值

| 作用域       | 说明                    |
| --------- | --------------------- |
| singleton | 默认，单例                 |
| request   | 调用获取实例方法时，每次请求创建一个实例  |
| session   | 调用获取实例方法时，每次会话创建一个实例  |
| prototype | 调用获取实例方法时，每次都创建一个新的实例 |

```xml
	<bean id="dome5" class="example.spring.beans.Dome" scope="prototype">
        <property name="name" value="张高峰"></property>
        <property name="message" value="测试"></property>
    </bean>
```

## 自动装配

自动装配使用bean标签的autowrie属性，该属性提供了四个值

| 值           | 作用                                       |
| ----------- | ---------------------------------------- |
| byName      | 根据属性名字自动装配                               |
| byType      | 根据属性类型自动装配，当根据类型自动装配时，如果上下文中有两个同一类型的bean时，会报异常。 |
| constructor | 根据构造函数自动装配                               |
| default     | 默认自动装配                                   |
| no          | 不执行自动装配                                  |

```xml
	<bean id="domeService" class="example.spring.service.impl.DomeServiceImpl" 							autowire="byName">
        <property name="dome" ref="dome"></property>
    </bean>
```

在这里使用自动装配的功能，当只需要个别属性需要自动装配的时候，就显的不够灵活了。

## 初始化方法，销毁方法

当加载Spring容器的时候，会自动创建所有bean实例。每个bean被创建后，都可以设置初始化方法和销毁的时候调用的方法，使用bean标签的**init-method**,**destory-method**属性

```xml
	<bean id="dome" class="example.spring.beans.Dome" init-method="a" destroy-method="b">
        <property name="name" value="${uname}"></property>
        <property name="message" value="${pwd}"></property>
    </bean>
```

## 后置处理器

spring提供的后置处理器，允许在调用初始化方法前后对Bean进行额外的处理。

比如：当spring容器实例化Dome对象完成后，调用初始化方法的前后，对Dome对象进行正确性判断，或者改变Dome对象的值等。

后置处理器，不是针对某一个bean的，是真多spring容器中的所有bean，都要经过后置处理器的处理

可以通过实现BeanPostProcessor接口创建一个后置处理器

```java
package example.spring.postProcessor;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;

public class MyDomePostProcessor implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {

        System.out.println("执行"+beanName+" 对象初始化方法之前");
        return bean;

    }

    /**
     *
     * @param bean 初始化的类
     * @param beanName 初始化对象的名字
     * @return
     * @throws BeansException
     */
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("执行"+beanName+" 对象初始化方法之后");
        return bean;
    }
}

```

在spring配置文件中配置后置处理器的bean

```xml
<!-- 后置处理器 -->
    <bean id="domePostProcessor" class="example.spring.postProcessor.MyDomePostProcessor"></bean>
```

测试

```java
		package example.spring.test;


import example.spring.beans.Dome;
import example.spring.service.DomeService;
import example.spring.service.impl.DomeServiceImpl;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class DomeTest {

    public static void main(String[] args) {

        //加载xml，获取spring上下文容器
        //当初始化spring容器的时候，就会创建好所有的单例对象
        ApplicationContext ap = new  ClassPathXmlApplicationContext("config/application.xml");
        Dome d1 = (Dome) ap.getBean("dome");


    }
}

```

输出

>    create Dome
>    执行dome 对象初始化方法之前
>    class create
>    执行dome 对象初始化方法之后

