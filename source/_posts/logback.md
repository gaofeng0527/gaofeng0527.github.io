---
title: Logback日志框架
date: 2017-12-13 11:42:49
tags: [java]
---

# 简介

logback和log4j是同一个人写的。logback重写了内核，比log4j更加强大，性能更好，因此log4j也可以被logback替代。

slf4j这个jar只是提供了一些接口，供其他日志框架实现。类似于jdbc一样。通俗的讲，slf4j告诉系统，你通过我可以使用任何实现我接口的日志系统。允许最终用户在部署系统时使用其所希望的日子System，只需要更换jar包即可。

因此建议logback和slf4j配合使用.

<!-- more -->

# 入门

## 导入jar依赖

logback分三个模块core（基础模块），classic（是log4j改进版，并且实现了slf4j接口,依赖core模块），access（与servlet容器集成，提供http访问记录功能）。

这里我们主要用到slf4j-api,logback-classic,logback-core

maven

```xml
<properties>
	<!-- base setting -->
	<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
	<project.build.locales>zh_CN</project.build.locales>
	<project.build.jdk>1.7</project.build.jdk>
	<plugin.maven-compiler>3.1</plugin.maven-compiler>
	<!-- lib versions -->
	<slf4j.version>1.7.21</slf4j.version>
	<logback.version>1.2.3</logback.version>
</properties>
<dependency>
	<groupId>org.slf4j</groupId>
	<artifactId>slf4j-api</artifactId>
	<version>${slf4j.version}</version>
</dependency>
<dependency>
	<groupId>ch.qos.logback</groupId>
	<artifactId>logback-classic</artifactId>
	<version>${logback.version}</version>
</dependency>
<dependency>
	<groupId>ch.qos.logback</groupId>
	<artifactId>logback-core</artifactId>
	<version>${logback.version}</version>
</dependency>
```

>    这里有个地方需要注意一下，当我们在maven仓库中拷贝`logback-classic`依赖时，会有`<scope>test</scope>`.  scope为test表示依赖项目仅仅参与测试相关的工作，包括测试代码的编译，执行。比较典型的如junit。[参考(maven secope详解)](http://blog.csdn.net/kimylrong/article/details/50353161)

## 简单应用

```java
package com.cack.logger;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HelloWord1 {

	public static void main(String[] args) {
		Logger logger = LoggerFactory.getLogger(HelloWord1.class);
		logger.debug("hello word");
	}

}
```

控制台输出

`13:22:30.228 [main] DEBUG com.cack.logger.HelloWord1 - hello word`

上面的例子可以看到，通过LoggerFactory可以获取一个logger，并且用logger的debug方法记录日子。

这里都是用到的slf4j的类。看不出对logback的依赖。实际上输入内容是有slf4j的实现类，也就是logback-classic包中的实现类输出的。

当使用logback输出日志的时候，它会默认去找logback-test.xml配置文件，如果没找到，会接着去找logback.xml配置文件，如果还没找到，就会使用默认的配置了。默认的配置是通过控制台输出。

也可以通过下面的例子看出来这个流程：

```java
package com.cack.logger;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import ch.qos.logback.classic.LoggerContext;
import ch.qos.logback.core.util.StatusPrinter;

public class HelloWord1 {

	public static void main(String[] args) {
		Logger logger = LoggerFactory.getLogger(HelloWord1.class);
		logger.debug("hello word");
		LoggerContext lc = (LoggerContext) LoggerFactory.getILoggerFactory();
		StatusPrinter.print(lc);
	}
}
```

>    13:28:08.792 [main] DEBUG com.cack.logger.HelloWord1 - hello word
>    13:28:08,744 |-INFO in ch.qos.logback.classic.LoggerContext[default] - Could NOT find resource [logback-test.xml]
>    13:28:08,744 |-INFO in ch.qos.logback.classic.LoggerContext[default] - Could NOT find resource [logback.groovy]
>    13:28:08,744 |-INFO in ch.qos.logback.classic.LoggerContext[default] - Could NOT find resource [logback.xml]
>    13:28:08,748 |-INFO in ch.qos.logback.classic.BasicConfigurator@6d5380c2 - Setting up default configuration.

## Logger,Appender,Layout

Logback建立在这三个主要的类上，这三个类协同合作可以帮助使用者根据消息类型和消息级别输出日志，还可以在程序运行期间，控制消息的输出格式和输出目的地。

### Logger 上下文

Logger主要日志输出类，负责日子的输出工作，可以被分级：TRACE，DEBUG，INFO，WARN，ERROR。如果Logger没有被定义级别，他会从他父类继承。

### Appender 输出器

主要用于控制日志输出目的地，目前有控制台，文件，远程套接字服务器，数据库，jms，远程UNIX Syslog守护进程等多种appender。默认是控制台appender。

可以为一个logger设置多个appender。

### Layout格式化

Layout 负责根据用户意愿对记录请求进行格式化，appender 负责将格式化后的输出发送到目的地。PatternLayout 是标准 logback 发行包的一部分，允许用户按照类似于 C 语言的 printf 函数的转换模式设置输出格式。

## 配置详解

### **<configuration>**包含的属性

>    1.   scan:当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true.
>    2.   scanPeriod：设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。当scan为true时，此属性生效。默认的时间间隔为1分钟.
>    3.   debug：当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。

```xml
<configuration scan="true" scanPeriod="60 second" debug="true">
</configuration>
```



### **<property>** 设置变量

用来定义变量值的标签，该标签有两个属性`name`,`value`,定义的变量会存在logger的上下文中，可以使用${} 来使用变量

### **<contextName>**上下文名称

设置上下文名称，用来区分不同应用程序的输出，一旦设置不允许修改。下面结合`<property>`标签设置上下文

```xml
<configuration scan="true" debug="true">
  <property name="cname" value="myApp"></property>
  <contextName>${cname}</contextName> 
  <!-- 上面两句等同于  <contextName>myApp</contextName>-->
</configuration>
```

### **<timestamp>** 获取时间戳

可以结合创建日志文件的时候作为文件名使用

```xml
<timestamp key="bySecond" datePattern="yyyyMMdd'T'HHmmss"></timestamp>
```

### **<logger>**

用来设置某一个包或者具体的某一个类的日志打印级别、以及指定`<appender>`。`<logger>`仅有一个name属性，一个可选的level和一个可选的additivity属性。

>    1.   name：用来指定受此logger约束的某一个包或者具体的某一个类。
>    2.   level：用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF，还有一个特殊值INHERITED或者同义词NULL，代表强制执行上级的级别。如果未设置此属性，那么当前logger将会继承上级的级别。
>    3.   additivity：是否向上级logger传递打印信息。默认是true。

`<logger>`可以包含零个或多个`<appender-ref>`元素，标识这个appender将会添加到这个logger。

当logger内没有配置appender的时候，本logger是不输出任何内容的。如果additivity设置为true的话，会把打印信息默认向上传输，父类配置了appender的话，就会输出日志信息了。

### **<root>**

也是`<logger>`元素，但是它是根logger。只有一个level属性，应为已经被命名为”root”.

>    1.   level：用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF，不能设置为INHERITED或者同义词NULL。默认是DEBUG。

`<root>`可以包含零个或多个`<appender-ref>`元素，标识这个appender将会添加到这个logger。

### **<appender>**详解

`<appender>`是`<configuration>`的子节点，是负责写日志的组件。`<appender>`有两个必要属性`name`和`class`。`name`指定`appender`名称，`class`指定`appender`的全限定名。

#### ConsoleAppender

把日志打印到控制台，有以下两个节点

>    1.   `<encoder>`:对日志进行格式化。encoder有个默认的layout `PatternLayoutEncoder`
>    2.   `<target>`：字符串 System.out 或者 System.err ，默认 System.out .

```xml
<configuration>  
  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">  
    <encoder>  
      <pattern>%-4relative [%thread] %-5level %logger{35} - %msg %n</pattern>  
    </encoder>  
  </appender>  
 
  <root level="DEBUG">  
    <appender-ref ref="STDOUT" />  
  </root>  
</configuration>
```

#### FileAppender

把日志添加到文件，有以下子节点：

>    1.   `<file>`：被写入的文件名，可以是相对目录，也可以是绝对目录，如果上级目录不存在会自动创建，没有默认值。
>    2.   `<append>`：如果是 true，日志被追加到文件结尾，如果是 false，清空现存文件，默认是true。
>    3.   `<encoder>`：对记录事件进行格式化。
>    4.   `<prudent>`：如果是 true，日志会被安全的写入文件，即使其他的FileAppender也在向此文件做写入操作，效率低，默认是 false。

```xml
<configuration>  
  <appender name="FILE" class="ch.qos.logback.core.FileAppender">  
    <file>testFile.log</file>  
    <append>true</append>  
    <encoder>  
      <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>  
    </encoder>  
  </appender>  
 
  <root level="DEBUG">  
    <appender-ref ref="FILE" />  
  </root>  
</configuration>
```

#### RollingFIleAppender

滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件。有以下子节点：

>    1.   `<file>`：被写入的文件名，可以是相对目录，也可以是绝对目录，如果上级目录不存在会自动创建，没有默认值。
>    2.   `<append>`：如果是 true，日志被追加到文件结尾，如果是 false，清空现存文件，默认是true。
>    3.   `<encoder>`：对记录事件进行格式化。
>    4.   `<rollingPolicy>`:当发生滚动时，决定 RollingFileAppender 的行为，涉及文件移动和重命名。
>    5.   `<triggeringPolicy >`: 告知 RollingFileAppender 何时激活滚动。
>    6.   `<prudent>`：当为true时，不支持FixedWindowRollingPolicy。支持TimeBasedRollingPolicy，但是有两个限制，1不支持也不允许文件压缩，2不能设置file属性，必须留空。

常用的policy，`TimeBasedRollingPolicy`根据时间节点滚动策略，`SizeBasedTriggeringPolicy`： 查看当前活动文件的大小，如果超过指定大小会告知RollingFileAppender 触发当前活动文件滚动。只有一个节点`<maxFileSize>`:这是活动文件的大小，默认值是10MB。

```xml
<configuration>   
  <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">   
 
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">   
      <fileNamePattern>logFile.%d{yyyy-MM-dd}.log</fileNamePattern>   
      <maxHistory>30</maxHistory>    
    </rollingPolicy>   
 
    <encoder>   
      <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>   
    </encoder>   
  </appender>    
 
  <root level="DEBUG">   
    <appender-ref ref="FILE" />   
  </root>   
</configuration>
```

另外还有SocketAppender、SMTPAppender、DBAppender、SyslogAppender、SiftingAppender，并不常用，这些就不在这里讲解了，大家可以参考官方文档。当然大家可以编写自己的Appender。

### **<encoder>**

负责两件事，一是把日志信息转换成字节数组，二是把字节数组写入到输出流。
目前PatternLayoutEncoder 是唯一有用的且默认的encoder ，有一个`<pattern>`节点，用来设置日志的输入格式。使用“%”加“转换符”方式，如果要输出“%”，则必须用“\”对“\%”进行转义。

```xml
<encoder>   
   <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>   
</encoder>
```

格式修饰符，与转换符共同使用：
可选的格式修饰符位于“%”和转换符之间。第一个可选修饰符是左对齐 标志，符号是减号“-”；接着是可选的最小宽度 修饰符，用十进制数表示。如果字符小于最小宽度，则左填充或右填充，默认是左填充（即右对齐），填充符为空格。如果字符大于最小宽度，字符永远不会被截断。最大宽度 修饰符，符号是点号”.”后面加十进制数。如果字符大于最大宽度，则从前面截断。点符号“.”后面加减号“-”在加数字，表示从尾部截断。

例如：%-4relative 表示，将输出从程序启动到创建日志记录的时间 进行左对齐 且最小宽度为4。

## 完整的logback.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--
-scan:当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true
-scanPeriod:设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。
-           当scan为true时，此属性生效。默认的时间间隔为1分钟
-debug:当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。
-
- configuration 子节点为 appender、logger、root
-->
<configuration scan="true" scanPeriod="60 second" debug="false">
 
    <!-- 负责写日志,控制台日志 -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
 
        <!-- 一是把日志信息转换成字节数组,二是把字节数组写入到输出流 -->
        <encoder>
            <Pattern>[%d{yyyy-MM-dd HH:mm:ss.SSS}] [%5level] [%thread] %logger{0} %msg%n</Pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>
 
    <!-- 文件日志 -->
    <appender name="DEBUG" class="ch.qos.logback.core.FileAppender">
        <file>debug.log</file>
        <!-- append: true,日志被追加到文件结尾; false,清空现存文件;默认是true -->
        <append>true</append>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <!-- LevelFilter: 级别过滤器，根据日志级别进行过滤 -->
            <level>DEBUG</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
        <encoder>
            <Pattern>[%d{yyyy-MM-dd HH:mm:ss.SSS}] [%5level] [%thread] %logger{0} %msg%n</Pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>
 
    <!-- 滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件 -->
    <appender name="INFO" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <File>info.log</File>
 
        <!-- ThresholdFilter:临界值过滤器，过滤掉 TRACE 和 DEBUG 级别的日志 -->
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>INFO</level>
        </filter>
 
        <encoder>
            <Pattern>[%d{yyyy-MM-dd HH:mm:ss.SSS}] [%5level] [%thread] %logger{0} %msg%n</Pattern>
            <charset>UTF-8</charset>
        </encoder>
 
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 每天生成一个日志文件，保存30天的日志文件
            - 如果隔一段时间没有输出日志，前面过期的日志不会被删除，只有再重新打印日志的时候，会触发删除过期日志的操作。
            -->
            <fileNamePattern>info.%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>30</maxHistory>
            <TimeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </TimeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
    </appender >
 
    <!--<!– 异常日志输出 –>-->
    <!--<appender name="EXCEPTION" class="ch.qos.logback.core.rolling.RollingFileAppender">-->
        <!--<file>exception.log</file>-->
        <!--<!– 求值过滤器，评估、鉴别日志是否符合指定条件. 需要额外的两个JAR包，commons-compiler.jar和janino.jar –>-->
        <!--<filter class="ch.qos.logback.core.filter.EvaluatorFilter">-->
            <!--<!– 默认为 ch.qos.logback.classic.boolex.JaninoEventEvaluator –>-->
            <!--<evaluator>-->
                <!--<!– 过滤掉所有日志消息中不包含"Exception"字符串的日志 –>-->
                <!--<expression>return message.contains("Exception");</expression>-->
            <!--</evaluator>-->
            <!--<OnMatch>ACCEPT</OnMatch>-->
            <!--<OnMismatch>DENY</OnMismatch>-->
        <!--</filter>-->
 
        <!--<triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">-->
            <!--<!– 触发节点，按固定文件大小生成，超过5M，生成新的日志文件 –>-->
            <!--<maxFileSize>5MB</maxFileSize>-->
        <!--</triggeringPolicy>-->
    <!--</appender>-->
 
    <appender name="ERROR" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>error.log</file>
 
        <encoder>
            <Pattern>[%d{yyyy-MM-dd HH:mm:ss.SSS}] [%5level] [%thread] %logger{0} %msg%n</Pattern>
            <charset>UTF-8</charset>
        </encoder>
 
        <!-- 按照固定窗口模式生成日志文件，当文件大于20MB时，生成新的日志文件。
        -    窗口大小是1到3，当保存了3个归档文件后，将覆盖最早的日志。
        -    可以指定文件压缩选项
        -->
        <rollingPolicy class="ch.qos.logback.core.rolling.FixedWindowRollingPolicy">
            <fileNamePattern>error.%d{yyyy-MM}(%i).log.zip</fileNamePattern>
            <minIndex>1</minIndex>
            <maxIndex>3</maxIndex>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
    </appender>
 
    <!-- 异步输出 -->
    <appender name ="ASYNC" class= "ch.qos.logback.classic.AsyncAppender">
        <!-- 不丢失日志.默认的,如果队列的80%已满,则会丢弃TRACT、DEBUG、INFO级别的日志 -->
        <discardingThreshold >0</discardingThreshold>
        <!-- 更改默认的队列的深度,该值会影响性能.默认值为256 -->
        <queueSize>512</queueSize>
        <!-- 添加附加的appender,最多只能添加一个 -->
        <appender-ref ref ="ERROR"/>
    </appender>
 
    <!--
    - 1.name：包名或类名，用来指定受此logger约束的某一个包或者具体的某一个类
    - 2.未设置打印级别，所以继承他的上级<root>的日志级别“DEBUG”
    - 3.未设置additivity，默认为true，将此logger的打印信息向上级传递；
    - 4.未设置appender，此logger本身不打印任何信息，级别为“DEBUG”及大于“DEBUG”的日志信息传递给root，
    -  root接到下级传递的信息，交给已经配置好的名为“STDOUT”的appender处理，“STDOUT”appender将信息打印到控制台；
    -->
    <logger name="ch.qos.logback" />
 
    <!--
    - 1.将级别为“INFO”及大于“INFO”的日志信息交给此logger指定的名为“STDOUT”的appender处理，在控制台中打出日志，
    -   不再向次logger的上级 <logger name="logback"/> 传递打印信息
    - 2.level：设置打印级别（TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF），还有一个特殊值INHERITED或者同义词NULL，代表强制执行上级的级别。
    -        如果未设置此属性，那么当前logger将会继承上级的级别。
    - 3.additivity：为false，表示此logger的打印信息不再向上级传递,如果设置为true，会打印两次
    - 4.appender-ref：指定了名字为"STDOUT"的appender。
    -->
    <logger name="com.weizhi.common.LogMain" level="INFO" additivity="false">
        <appender-ref ref="STDOUT"/>
        <!--<appender-ref ref="DEBUG"/>-->
        <!--<appender-ref ref="EXCEPTION"/>-->
        <!--<appender-ref ref="INFO"/>-->
        <!--<appender-ref ref="ERROR"/>-->
        <appender-ref ref="ASYNC"/>
    </logger>
 
    <!--
    - 根logger
    - level:设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF，不能设置为INHERITED或者同义词NULL。
    -       默认是DEBUG。
    -appender-ref:可以包含零个或多个<appender-ref>元素，标识这个appender将会添加到这个logger
    -->
    <root level="DEBUG">
        <appender-ref ref="STDOUT"/>
        <!--<appender-ref ref="DEBUG"/>-->
        <!--<appender-ref ref="EXCEPTION"/>-->
        <!--<appender-ref ref="INFO"/>-->
        <appender-ref ref="ASYNC"/>
    </root>
</configuration>
```



