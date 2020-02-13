---
title: IDEA springboot 修改后自动编译部署
date: 2020-02-13 11:24:41
tags: springboot
---

# 需求

​	在开发阶段，会经常修改代码，每次修改后都要重启应用才能看到效果，非常麻烦。

# 解决

1. 引入依赖

   ```xml
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-devtools</artifactId>
               <optional>true</optional> <!-- 为true的时候才会生效 -->
               <scope>runtime</scope>
           </dependency>
   
   <!-- 引入插件 -->
   	<build>
           <plugins>
               <plugin>
                   <groupId>org.springframework.boot</groupId>
                   <artifactId>spring-boot-maven-plugin</artifactId>
                   <configuration>
                       <fork>true</fork>
                       <addResources>true</addResources>
                   </configuration>
               </plugin>
           </plugins>
       </build>
   ```

   

2. 修改IDEA配置

   1.打开Settings(Ctrl+Shift+A)---》Build,Execution,Deployment --->compiler  

   2.勾选 Build project automatically 

   3.apply

3. 重启IDEA

4. 测试