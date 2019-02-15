---
title: spring aop
date: 2016-11-28 11:56:21
tags: spring
---

# AOP简介

想必大家对OOP都已经很熟悉了，其实AOP就是对OOP的一个有益补充。万变不离其中，我理解的AOP其实也是用面向对象的思想编写好的一个业务功能，只是这个业务功能是环绕在同一个项目中的很多甚至所有的其他核心业务功能的组件。帮助我们解决系统中单独的且又和每个核心功能有牵扯的某一方面问题，比如事务控制，日志，安全等。
<!-- more -->
讲一个生活中通俗点的故事：一个歌手在录制专辑，但是歌手的主要职责就是用心把歌唱好，可是他只唱歌也出不了专辑影像，那么就需要一个人来录制。当歌手开始唱歌之前，录制人员打开录音机；歌手唱完了，录制人员关闭录音机；当歌手在演唱过程中突发状况了，录制人员也可以关闭录音机。有人问了，这不是两个对象相互合作的吗？那如果是一家唱片公司，那么多歌手，假设歌手们都不是同时录音的，每个人都要配置一个录音团队吗，显然不是一个好的选择。怎么使每个歌手在唱歌的时候，都能有录音团度跟上工作呢，这就是AOP重点解决的问题。（例子有点糙，反正我是明白的，哈哈哈）

# 基于XML配置AOP

## 添加依赖

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aop</artifactId>
    <version>4.1.1.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.8.1</version>
</dependency>
```



## 创建一个通知

即把处理核心业务之外的另外的操作，又是处理同一问题领域的功能，比如给核心操作添加日志的功能，即一个切面。

```java
package example.spring.aop;

public class LoggerAspect {

    public void doBefore(){
        System.out.println("执行方法前");
    }

    public void doAfter(){
        System.out.println("执行方法之后");
    }

    public void runException(){
        System.out.println("执行抛异常的时候");
    }

    public void onAround(){
        System.out.println("方法执行中");
    }
}

```

## 添加核心业务

Service接口

```java
package example.spring.service;
public interface DomeService {

    public void save();
}
```

Impl实现类

```java
package example.spring.service.impl;

import example.spring.service.DomeService;

public class DomeServiceImpl implements DomeService {

  @Override
    public void save() {
        System.out.println("mysql save one Dome success");
    }
}
```



## 在spring配置文件中配置AOP

```xml

    <!-- 使用 XML 配置AOP -->
    <aop:config>
        <!-- 创建切面配置 -->
        <aop:aspect ref="loggerAspect">
            <!-- 创建一个切点 -->
            <aop:pointcut id="methodPointCut" expression="execution(* example.spring.service.*.*(..))"></aop:pointcut>
            <!-- 织入 -->
            <aop:before method="doBefore" pointcut-ref="methodPointCut"></aop:before>
            <aop:after method="doAfter" pointcut-ref="methodPointCut"></aop:after>
            <aop:around method="onAround" pointcut-ref="methodPointCut"></aop:around>
            <aop:after-throwing method="runException" pointcut-ref="methodPointCut"></aop:after-throwing>
        </aop:aspect>
    </aop:config>
```

## 测试

```java
package example.spring.test;

import example.spring.service.DomeService;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class AspectTest {

    public static void main(String[] args) {
        ApplicationContext ap = new ClassPathXmlApplicationContext("config/application.xml");
        DomeService dservice = (DomeService) ap.getBean("domeService");
        dservice.save();
    }
}

```

## 结果

>    Exception in thread "main" org.springframework.beans.factory.BeanNotOfRequiredTypeException: Bean named 'domeService' is expected to be of type 'example.spring.service.impl.DomeServiceImpl' but was actually of type 'com.sun.proxy.$Proxy4'
>
>    	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:384)
>    	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:205)
>    	at org.springframework.context.support.AbstractApplicationContext.getBean(AbstractApplicationContext.java:1091)
>    	at example.spring.test.AspectTest.main(AspectTest.java:11)

报异常了，为什么呢，这是因为AOP采用了代理模式，通过配置创建核心业务对象的代理来使用。

proxy-target-class属性值决定是基于接口的还是基于类的代理被创建。首先说明下proxy-target-class="true"和proxy-target-class="false"的区别，为true则是基于类的代理将起作用（需要cglib库），为false，则标准的JDK 基于接口的代理将起作用。
proxy-target-class在spring事务、aop、缓存这几块都有设置，其作用都是一样的。

我们上面采用的是接口代理，aop:config的配置默认是类注入代理模式，因此类型错误了。

我们把aop那一块的配置加上 proxy-target-class属性

```xml
<aop:config proxy-target-class="true">
        <!-- 创建切面配置 -->
        <aop:aspect ref="loggerAspect">
            <!-- 创建一个切点 -->
            <aop:pointcut id="methodPointCut" expression="execution(* example.spring.service.impl.*.*())"></aop:pointcut>
            <!-- 织入 -->
            <aop:before method="doBefore" pointcut-ref="methodPointCut"></aop:before>
            <aop:after method="doAfter" pointcut-ref="methodPointCut"></aop:after>

        </aop:aspect>
    </aop:config>
```



测试结果

>    执行方法前
>    mysql save one Dome success
>    执行方法之后



# 基于注解配置AOP

##  配置文件开启AOP注解

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">


    <!-- 开启aop注解模式，设置类代理模式 -->
    <aop:aspectj-autoproxy proxy-target-class="false"></aop:aspectj-autoproxy>
    <bean id="customerService" class="example.spring.service.CustomerService"></bean>
  	<!-- 通知类，即使加上注解，也需要在spring容器中注册 -->
    <bean id="callAdvice" class="example.spring.aop.CallAdvice"></bean>
</beans>
```

## 通知类

```java
package example.spring.aop;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;

@Aspect
public class CallAdvice {
    public static int steps = 1;


    @Before("execution(* example.spring.service.*.*())")
    public void doBefore(JoinPoint jp) {
//        System.out.println(jp.toString());
//        System.out.println(jp.getTarget().toString());
//        System.out.println(jp.getTarget().getClass().getName());
//        System.out.println(jp.getSignature().toString());
//        System.out.println(jp.getSignature().getName());
        System.out.println(jp.getTarget().getClass().getName() + "方法执行前");

    }

    @After("execution(* example.spring.service.*.*())")
    public void doAfter() {
        System.out.println("方法执行后");
    }

    @Around("execution(* example.spring.service.*.*())")
    public Object onAround(ProceedingJoinPoint pjp) throws Throwable {
        String className = pjp.getTarget().getClass().getName();
        String methodName = pjp.getSignature().getName();
        int sn = steps++;
        System.out.println(String.format("[%d: METHOD BEFORE]: %s.%s()", sn, className, methodName));
        long time = System.currentTimeMillis();
        Object retVal = pjp.proceed();
        time = System.currentTimeMillis() - time;
        System.out.println(String.format("[%d: METHOD  AFTER]: %s.%s() : %dms\n", sn, className, methodName, time));
        return retVal;

    }
}

```

## 核心业务类

```java
package example.spring.service;

public class CustomerService {

    public void save(){
        System.out.println("保存客户信息");
    }
}

```

## 测试

```java
package example.spring.test;

import example.spring.service.CustomerService;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Customer {

    public static void main(String[] args) {
        ApplicationContext ap = new ClassPathXmlApplicationContext("config/application-aop.xml");
        CustomerService cs = ap.getBean("customerService",CustomerService.class);
        cs.save();
    }
}

```

结果：

>    [1: METHOD BEFORE]: example.spring.service.CustomerService.save()
>    example.spring.service.CustomerService方法执行前
>    保存客户信息
>    [1: METHOD  AFTER]: example.spring.service.CustomerService.save() : 19ms
>
>    方法执行后



