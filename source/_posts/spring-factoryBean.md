---
title: spring factoryBean
date: 2018-06-29 14:33:46
tags: spring
---

在spring中，创建bean的方式有三种，前面已经介绍了使用全类名的方式，在applicationContext.xml中配置bean。下面介绍另外两种工厂方法和factory

# 工厂方法

## 静态工厂方法

静态工厂是用来创建bean的（可以通过spring配置，把工厂配置好，我们直接从工厂里获取bean）

<!-- more -->

### 先创建一个工厂类

```java
package example.spring.factory;

import example.spring.beans.Dome;

public class DomeFactory {

    public static Dome getDomeInstance(){
        Dome dome = new Dome();
        //这里可以添加复杂的逻辑
        dome.setName("factory");
        dome.setMessage("create object");
        return dome;
    }
}

```

### 声明工厂bean

要声明通过静态工厂方法创建的**bean**，需要在bean标签的class属性里制定拥有该工厂方法的类。同时在factory-method属性里制定创建对象的工厂方法名称，最后使用<constrctor-arg>标签为该方法参数

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:util="http://www.springframework.org/schema/util"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <bean id="dateFormat" class="java.text.DateFormat" factory-method="getDateInstance">
        <constructor-arg value="2"></constructor-arg>
    </bean>

    <bean id="domeFactory" class="example.spring.factory.DomeFactory" factory-method="getDomeInstance"></bean>
</beans>
```

### 测试

```java
package example.spring.test;

import example.spring.beans.Dome;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import java.text.DateFormat;

public class DateFormatTest {


    public static void main(String[] args) {
        ApplicationContext ac = new ClassPathXmlApplicationContext("config/application.xml");
        DateFormat df = (DateFormat) ac.getBean("dateFormat");
        System.out.println(df);

        Dome domeFactory = (Dome) ac.getBean("domeFactory");
        System.out.println(domeFactory);
    }
}

```

### 输出

>    java.text.SimpleDateFormat@ef7951d7
>    Dome{name='factory', message='create object', adds=null}

# FactoryBean

FactoryBean 是用来创建 Bean 的工厂，实现 `FactoryBean 接口` 的类就可以用来创建其他 Bean 了，在 Bean Configuration File 里的 bean标签 里定义要生成的 Bean。

FactoryBean 命名规范：`Bean 的类名` 后跟着 `FactoryBean`，例如创建 Dome的 FactoryBean 的类名应该为 DomeFactoryBean。

如果 Bean 的创建不只是简单的 setter, constructor 注入，而是有其他的逻辑，这个时候就可以用 FactoryBean 来创建 Bean。

FactoryBean 是很常见的，例如 Spring 集成 MyBatis 时用 SqlSessionFactoryBean 来创建 SqlSession。

## 实现FactoryBean接口的类

```java
package example.spring.factory;

import example.spring.beans.Dome;
import org.springframework.beans.factory.FactoryBean;

public class DomeFactoryBean implements FactoryBean<Dome> {
    private String name;
    private String message;

    public void setName(String name) {
        this.name = name;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    @Override
    public boolean isSingleton() {
        return false;
    }

    @Override
    public Dome getObject() throws Exception {
        Dome dome = new Dome();
        //可以添加业务逻辑
        dome.setName(name);
        dome.setMessage(message);
        return dome;
    }

    @Override
    public Class<?> getObjectType() {
        return null;
    }
}

```

## 配置bean

和普通bean配置一样，只不过，他返回的是要生成的类，而不是这里配置的工厂类

```xml
<bean id="domeFactoryBean" class="example.spring.factory.DomeFactoryBean">
        <property name="name" value="tomBean"></property>
        <property name="message" value="factory create dome"></property>
    </bean>
```

## 测试

```java
package example.spring.test;

import example.spring.beans.Dome;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class DateFormatTest{


    public static void main(String[] args) {
        ApplicationContext ac = new ClassPathXmlApplicationContext("config/application.xml");
        Dome dome = (Dome) ac.getBean("domeFactoryBean");
        System.out.println(dome);

    }
}

```

结果

>Dome{name='tomBean', message='factory create dome', adds=null}

