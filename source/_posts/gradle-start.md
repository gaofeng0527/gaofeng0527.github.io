---
title: Gradle 入门
date: 2018-06-26 08:54:47
tags: Gradle
---

# 简介

记得初次写HelloWord级别的WEB应用时，是用MyEclipse写的，要经过项目---> 右键---> 导出---> 选择导出方式---> 部署到tomcat---> 启动tomcat等一系列的操作，才能在浏览器端看到我们写的HelloWord。初识Gradle是我的部门总监彪哥介绍给我们的，说用Gradle做项目自动化构建和部署，一个命令就搞定了。当时还想这么神奇，就让我们一起来学习一下吧。

# 下载

1.   先去官网下载[https://gradle.org/](https://gradle.org/)
2.   下载后解压到指定目录，配置环境变量**GRADLE_HOME**，值为${contentPath}/gradle-4.5.1/bin
3.   配置好环境变量，打开命令行输入`gradle -v` ，显示如下，即安装成功

   ```

------------------------------------------------------------
Gradle 4.5.1
------------------------------------------------------------

Build time:   2018-02-05 13:22:49 UTC
Revision:     37007e1c012001ff09973e0bd095139239ecd3b3

Groovy:       2.4.12
Ant:          Apache Ant(TM) version 1.9.9 compiled on February 2 2017
JVM:          1.8.0_101 (Oracle Corporation 25.101-b13)
OS:           Windows 7 6.1 amd64
   ```

