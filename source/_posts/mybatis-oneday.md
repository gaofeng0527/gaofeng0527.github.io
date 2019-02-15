---
title: 创建mybatis项目
date: 2017-04-27 11:50:51
tags: mybatis
---

# 搭建mybatis应用的步骤

前面已经简单介绍过了mybatis，今天详细的整理一下创建mybatis项目的详细步骤。数据库的操作基本就是增·删·改·查，其中又已查询最为多。

<!-- more -->

## 搭建一个完整的mybatis应用

1.使用maven创建一个java web项目，参考https://gaofeng0527.github.io/eclipse-create-Maven-Project/

2.在pom.xml配置文件中添加mybatis 和 mysql驱动器依赖

``` xml
		<!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
		<dependency>
			<groupId>org.mybatis</groupId>
			<artifactId>mybatis</artifactId>
			<version>3.4.4</version>
		</dependency>
		<!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>5.1.6</version>
		</dependency>	
```

3.创建mybatis-config.xml，mybatis的主要配置文件。主要设置数据库信息，映射文件注册，还有更多的设置，我们后面慢慢研究

``` xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<properties resource="config.properties">
		
	</properties>
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
	
  	<!-- 添加映射文件的注册 -->
	<mappers>
		<mapper resource="com/lms/cert/model/ClassUserMapper.xml" />
	</mappers>

</configuration>
```

4.假设我们已经在数据库中创建好了表，结构如下

``` sql
CREATE TABLE IF NOT EXISTS `t_class_user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `class_id` bigint(20) NOT NULL,
  `class_course_id` bigint(20) DEFAULT NULL,
  `user_id` bigint(20) NOT NULL,
  `card_id` bigint(20) DEFAULT NULL,
  `register_time` date NOT NULL,
  `register_end_time` date DEFAULT NULL,
  `score` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `FKAC106ADD159C7722` (`class_id`),
  KEY `FKAC106ADD4529535F` (`user_id`),
  KEY `FKAC106ADD2969E25` (`class_course_id`)
) ENGINE=InnoDB  DEFAULT CHARSET=gb2312 AUTO_INCREMENT=7428 ;
```

5.编写java类 ClassUser

``` java
package com.lms.cert.model;

import java.util.Date;

public class ClassUser {

	private Long id;
	private Long class_id;
	private Long class_course_id;
	private Long user_id;
	private Long card_id;
	private Date register_time;
	private Date register_end_time;
	private Integer score;

	public Long getId() {
		return id;
	}

	public void setId(Long id) {
		this.id = id;
	}

	public Long getClass_id() {
		return class_id;
	}

	public void setClass_id(Long class_id) {
		this.class_id = class_id;
	}

	public Long getClass_course_id() {
		return class_course_id;
	}

	public void setClass_course_id(Long class_course_id) {
		this.class_course_id = class_course_id;
	}

	public Long getUser_id() {
		return user_id;
	}

	public void setUser_id(Long user_id) {
		this.user_id = user_id;
	}

	public Long getCard_id() {
		return card_id;
	}

	public void setCard_id(Long card_id) {
		this.card_id = card_id;
	}

	public Date getRegister_time() {
		return register_time;
	}

	public void setRegister_time(Date register_time) {
		this.register_time = register_time;
	}

	public Date getRegister_end_time() {
		return register_end_time;
	}

	public void setRegister_end_time(Date register_end_time) {
		this.register_end_time = register_end_time;
	}

	public Integer getScore() {
		return score;
	}

	public void setScore(Integer score) {
		this.score = score;
	}

	@Override
	public String toString() {
		return "ClassUser [id=" + id + ", class_id=" + class_id + ", class_course_id=" + class_course_id + ", user_id="
				+ user_id + ", card_id=" + card_id + ", register_time=" + register_time + ", register_end_time="
				+ register_end_time + ", score=" + score + "]";
	}
	
}

```

6.创建ClassUserMapper.xml映射文件，要执行的数据库操作的sql都在该配置文件中编写。

``` xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.lms.cert.inter.ClassUserMapper">
	<!-- 下面的sql是不是看着很眼熟 -->
	<select id="selectClassUser" resultType="com.lms.cert.model.ClassUser">
		select * from t_class_user where id = #{id}
	</select>
	
</mapper>
```

7.编写测试类

``` java
package com.lms.cert.test;

import java.io.IOException;
import java.io.InputStream;
import java.util.Date;
import java.util.List;
import java.util.Map;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import com.lms.cert.inter.ClassUserMapper;
import com.lms.cert.model.ClassUser;

public class Test {

	public static void main(String[] args) {

		String resource = "mybatis-config.xml";
		SqlSessionFactory sessionFactory = null;
		SqlSession session = null;
		try {
          	//获取SqlSessionFactory对象
			InputStream in = Resources.getResourceAsStream(resource);
			sessionFactory = new SqlSessionFactoryBuilder().build(in);
			System.out.println(sessionFactory);
          	// 获取session
			session = sessionFactory.openSession();
          	// 根据映射文件中select标签下的id调用对应的sql
			ClassUser cu = session.selectOne("com.lms.cert.inter.ClassUserMapper.selectClassUser", 7401);
			if (null != cu) {
				System.out.println(cu);
			}
			
			
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} finally {
			if(null != session){
				session.close();
			}
			
		}

	}

}

```

8.输出结果

>    org.apache.ibatis.session.defaults.DefaultSqlSessionFactory@33a10788
>    ClassUser [id=7401, class_id=81, class_course_id=237, user_id=2, card_id=null, register_time=Sat Oct 08 00:00:00 CST 2016, register_end_time=null, score=0]

大功告成，是不是很好学！