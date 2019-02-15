---
title: Redis安装
date: 2018-09-28 16:05:44
tags: redis
---

# 开场白

今天正式开始接触和学习Redis，就从安装开始吧。

<!-- more -->

# 下载

redis官方网站redis.io，在官网中的zip包据说只支持linux系统（没有实际实验过）。后来问度娘，说是有个团队在维护这一个window可用的redis包，在此先行谢过。

下载地址：[https://github.com/MSOpenTech/redis/tags](https://github.com/MSOpenTech/redis/tags)

根据系统要求，下载 Redis-x64-3.2.100.zip

![20180609102114561](C:\Users\Administrator\Desktop\20180609102114561.png)

# 安装

我是安装在D盘，在D盘下创建一个redis目录，然后把压缩包内的文件解压到当前目录下。安装ok

# 测试

```doc
//启动
redis-server redis.windows.conf
```

如果不抱错，就算启动成功了。

```doc
//从新打开一个命令行  默认端口6379
redis-cli -h 127.0.0.1 -p 6379
//连接成功后，测试存储   peak 为key   gaofeng为value
set peak gaofeng   
//读取
get peak
```

没错，和你预想的结果一样，基本就是成功了。

# 配置服务

这时，如果关闭了，开启redis的命令行窗口，在进行存取，就没有反应了。是因为redis服务已经随着窗口的关闭，而停止了。

我们可以把redis作为一个服务安装到我们的操作系统中，就像最初学习SQL Server那会，需要去服务页面，先开启服务。

安装服务：

redis-server --service-install redis.windows.conf

>    [575384] 28 Sep 16:20:59.787 # Granting read/write access to 'NT AUTHORITY\NetworkService' on: "D:\redis" "D:\redis\"
>    [575384] 28 Sep 16:20:59.790 # Redis successfully installed as a service.

去服务页面，打开redis服务，进行测试，成功。

卸载服务：

redis-server.exe --service-uninstall

>    [579236] 28 Sep 16:26:28.519 # Redis service successfully uninstalled.

表示卸载成功。

启动服务：

redis-server --service-start

停止服务：

redis-server --service-stop