---
title: MyBatis简介
date: 2017-04-27 10:53:08
tags: mybatis
---



# MyBatis简介

MyBatis是一个轻量级的ORM（Object Relational Mapping）框架。支持普通的SQL查询，存储过程和高级映射的优秀持久层框架。消除了大部分JDBC代码；主要使用xml或者注解完成配置和原始映射。相比较Hibernate框架更加灵活。

<!-- more -->

# 入门

每个MyBatis的应用程序都以一个SqlSessionFactory对象的实例为核心，因为我们要从这个对象中获取SqlSession对象去操作数据库。SqlSessionFactory对象的实例可以通过SqlSessionFactoryBuilder对象来获得，该对象可以从XML配置文件

## 简单查询

1.创建mybatis-config.xml

``` xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<!-- 引入资源文件 -->
	<properties resource="config.properties">
		
	</properties>
  <!-- 配置数据库信息 -->
	<environments default="development">
		<environment id="development">
			<transactionManager type="JDBC" />
			<dataSource type="POOLED">
				<property name="driver" value="${driver}" />
				<property name="url" value="${url}" />
				<property name="username" value="${uname}" />
				<property name="password" value="" />
			</dataSource>
		</environment>
	</environments>
	<!-- 引入映射文件，后面会讲到-->
	<mappers>
		<mapper resource="com/lms/cert/model/ClassUserMapper.xml" />
		<mapper resource="com/lms/cert/model/CardBatchMapper.xml" />
		<mapper resource="com/lms/cert/model/CardMapper.xml" />
	</mappers>

</configuration>
```

2.加载配置文件，并获取SqlSessionFactory

``` java
		String resource = "mybatis-config.xml";
		SqlSessionFactory sessionFactory = null;
		SqlSession session = null;
		try {
			InputStream in = Resources.getResourceAsStream(resource);
			sessionFactory = new SqlSessionFactoryBuilder().build(in);
			System.out.println(sessionFactory);
			
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
```

>    org.apache.ibatis.session.defaults.DefaultSqlSessionFactory@33a10788



3.创建一个数据表映射文件xxxMapper.xml

``` xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- 这里的namespace命名空间需要注意，如果后面需要用到接口编程的时候，这里要使用创建的接口的全名，否则会报接口未注册的异常 -->
<mapper namespace="com.lms.cert.inter.ClassUserMapper">
	
	<resultMap type="com.lms.cert.model.ClassUser" id="resultListUser">
		<id column="id" property="id"/>
		<result column="class_id" property="class_id"/>
		<result column="class_course_id" property="class_course_id"/>
		<result column="card_id" property="card_id"/>
		<result column="register_time" property="register_time"/>
	</resultMap>
	
  <!-- 这里select元素的id，如果使用接口编程，此处id的值要与接口中的方法名保持一致 -->
	<select id="selectByTime" parameterType="java.util.Date" resultMap="resultListUser">
		<![CDATA[select * from t_class_user where register_time <= #{beginTime}]]>
	</select>
	
	<select id="selectClassUser" resultType="com.lms.cert.model.ClassUser">
		select * from t_class_user where id = #{id}
	</select>
	
</mapper>
```

4.通过sqlSessionFactory实例获取session，并进行一次查询操作

``` java
			session = sessionFactory.openSession();
			ClassUser cu = session.selectOne("com.lms.cert.inter.ClassUserMapper.selectClassUser", 7401);
			if (null != cu) {
				System.out.println(cu);
			}	
```

>    ClassUser [id=7401, class_id=81, class_course_id=237, user_id=2, card_id=null, register_time=Sat Oct 08 00:00:00 CST 2016, register_end_time=null, score=0]

记得查询完，要关闭session









