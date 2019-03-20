---
title: 初识MongoDB
date: 2019-03-15 16:41:33
tags: [mongoDB]
---

# MongoDB

非关系型数据库；分布式文档存储数据库。内容大部分以json的格式存储

在`MongoDB`中基本概念是 文档（document）、集合(collection)、数据库(database)、数据字段\域（field）

一个`MongoDB`中可以创建多个数据库

admin： 从权限的角度来看，这是"root"数据库。要是将一个用户添加到这个数据库，这个用户自动继承所有数据库的权限。一些特定的服务器端命令也只能从这个数据库运行，比如列出所有的数据库或者关闭服务器。
local: 这个数据永远不会被复制，可以用来存储限于本地单台服务器的任意集合
config: 当Mongo用于分片设置时，config数据库在内部使用，用于保存分片的相关信息。

MongoDB 的文档不需要设置相同的字段，并且相同的字段不需要相同的数据类型，这与关系型数据库有很大的区别，也是 MongoDB 非常突出的特点。

<!-- more -->

# 下载

windows 下载地址：https://www.mongodb.com/download-center/community?jmp=docs

选择对应的系统版本和安装方式，点击下载即可

# 安装

1.   点击安装文件，一路next
2.   在安装过程中，可以设置db存储路径和日志输出路径
3.   不要勾选`Install MongoDB Compass`，因为会联网安装，在国内的话，还是省省吧
4.   最后点击`Install`等着就行了

>    我当时安装完，去在安装页面设置的路径中查找我刚安装的MongoDB，结果一脸懵逼，这里只有看其他教程需要手动创建的db和log文件夹，bin文件目录呢？？？
>
>    试想，会不会给我装到C盘了，一看果然如此，在C盘的`Program Files`目录下。安装时没认真看选择路径，就先放这里吧

>    后来看了官方文档，当前版本4.0 是可以选择以服务形式安装，就会默认创建好`mongod.cfg`配置文件。并默认安装到C盘。
>
>    并且会默认创建一个`MongoDB`的服务，等安装成功后会自动启动。

# 运行

因为前面我们选择了自动安装服务的模式，因此可以去bin目录下直接运行`mongo.exe`

```shell
MongoDB shell version v4.0.6
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("4b452b86-6bff-4780-9920-286b6f1114dc") }
MongoDB server version: 4.0.6
Welcome to the MongoDB shell.
For interactive help, type "help".
For more comprehensive documentation, see
        http://docs.mongodb.org/
Questions? Try the support group
        http://groups.google.com/group/mongodb-user
Server has startup warnings:
2019-03-15T16:51:40.440+0800 I CONTROL  [initandlisten]
2019-03-15T16:51:40.440+0800 I CONTROL  [initandlisten] ** WARNING: Access control is not enabled for the database.
2019-03-15T16:51:40.440+0800 I CONTROL  [initandlisten] **          Read and write access to data and configuration is unrestricted.
2019-03-15T16:51:40.440+0800 I CONTROL  [initandlisten]
---
Enable MongoDB's free cloud-based monitoring service, which will then receive and display
metrics about your deployment (disk utilization, CPU, operation statistics, etc).

The monitoring data will be available on a MongoDB website with a unique URL accessible to you
and anyone you share the URL with. MongoDB may use this information to make product
improvements and to suggest MongoDB products and deployment options to you.

To enable free monitoring, run the following command: db.enableFreeMonitoring()
To permanently disable this reminder, run the following command: db.disableFreeMonitoring()
---
```

启动成功。

或者在浏览器中输入`localhost:27017` 页面显示

`It looks like you are trying to access MongoDB over HTTP on the native driver port.`

也表示启动成功

# 使用MongoDB

## 常用命令

1.   show dbs    显示数据库列表
2.   user dbname 进入某个库
3.   show collections   显示集合列表（关系型数据库中的表）
4.   help 查看MongoDB支持哪些命令
5.   db.help() 查看当前数据库支持哪些方法

## 创建 & 新增

数据库方面操作

1.   use db_name  切换数据库，如果不存在则自动创建
2.   db.getName() 查看当前数据库名字 可以简写：`db`
3.   db.stats() 查看当前数据库状态，包括名字，有多少个集合等
4.   db.dropDatabase() 删除当前数据库

集合操作

1.   db.createCollection('collection_name')  创建集合

     | 字段          | 类型   | 描述                                       |
     | ----------- | ---- | ---------------------------------------- |
     | capped      | 布尔   | （可选）如果为 true，则创建固定集合。固定集合是指有着固定大小的集合，当达到最大值时，它会自动覆盖最早的文档。**当该值为 true 时，必须指定 size 参数。** |
     | autoIndexId | 布尔   | （可选）如为 true，自动在 _id 字段创建索引。默认为 false。    |
     | size        | 数值   | （可选）为固定集合指定一个最大值（以字节计）。**如果 capped 为 true，也需要指定该字段。** |
     | max         | 数值   | （可选）指定固定集合中包含文档的最大数量。                    |

     >相当于关系型数据库中创建表
     >
     >```json
     >db.createCollection("mycol",{capped:true,autoIndexId:true,size:6142800,max:1000})
     >```
     >
     >​

2.   db.col.insert() 向集合中添加内容，如果集合不存在，会自动创建

     ```json
     db.user.insert({uname:"gaofeng"}) -- 添加一个名为user的集合，并添加一条数据
     ```

3.   db.col.drop()  可以删除集合

插入一条数据

1.    db.col.insert({username:"gaofeng",age:25,address:"北京"})  插入一条数据
2.   也可以声明一个对象：

>    ```json
>    > users = {uname:"lisi",age:25,address:"beijing"}
>    > users   -- 再次输入可以查看
>    { "uname" : "lisi", "age" : 25, "address" : "beijing" }
>    ```
>
>    db.col.insert(users) 进行插入
>
>    db.col.save(users) 也可以进行插入，当提供objectId的时候，就是更新操作了

# 更新

update() 和 save() 方法都可以完成更新操作

1.   db.col.update(`<query>`,`<update>`,{upsert:`<boolean>`,multi:`<boolean>`,writeConcern:`<document>`})

>    query : update的查询条件，例如要查询uname为gaofeng的，这里可以直接写{uname:"gaofeng"}
>
>    update : update的对象和一些更新的操作符，对应sql中set后面的内容
>
>    upsert : 可选，表示如果不存在更新的内容，是否插入新的文档，true为插入，false不插入，默认为false
>
>    multi : 可选，默认为false，只更新找到的第一条，如果设置为true，则更新所有符合条件的记录
>
>    writeConcern : 可选，抛出异常的级别
>
>    ```json
>    > db.user.update({uname:"gaofeng"},{$set:{age:15}},{multi:true})
>    WriteResult({ "nMatched" : 2, "nUpserted" : 0, "nModified" : 2 })
>    ```
>
>    更新了两条记录

2.   db.col.save(`<document>`,writeConcern:`<document>`)

     前面介绍过，save一条数据时，如果带有objectId值，并且该值在集合中存在，就会自动执行更新操作。如果没有，则执行插入操作。

# 删除

删除使用`db.col.remove()`

>    语法：db.col.remove(`<query>`,{justOne:`<boolean>`,writeConcern:`<document>`})
>
>    query: 可选，删除的赛选条件，相当于sql中where后面的内容
>
>    justOne：可选，如果为true或1，则只删除符合条件的第一条文档，反之删除所有，默认为false；

实例：

```json
db.col.remove({uname:'gaofeng'},{justOne:true})  //删除名字为gaofeng的第一条文档

db.col.remove({})   //删除所有数据
```

# 查询

语法：

>    db.col.find(query,projection)
>
>    query : 可选，查询条件
>
>    projection ： 使用投影操作符指定返回的键，如果要返回所有键值，忽略这个参数
>
>    db.col.find().pretty() 可以格式化显示查询结果
>
>    ```json
>    > db.user.find({uname:'gaofeng'}).pretty()
>    {
>            "_id" : ObjectId("5c8b716e4a5c57c4ada08ab6"),
>            "uname" : "gaofeng",
>            "age" : 15
>    }
>    {
>            "_id" : ObjectId("5c8b72bf4a5c57c4ada08ab8"),
>            "uname" : "gaofeng",
>            "age" : 15
>    }
>    ```
>
>    

MongoDB 于 SQL中的语法比较

| 操作    | 格式           | 范例                                       | RDBMS中的类似语句         |
| ----- | ------------ | ---------------------------------------- | ------------------- |
| 等于    | `{:`}        | `db.col.find({"by":"菜鸟教程"}).pretty()`    | `where by = '菜鸟教程'` |
| 小于    | `{:{$lt:}}`  | `db.col.find({"likes":{$lt:50}}).pretty()` | `where likes < 50`  |
| 小于或等于 | `{:{$lte:}}` | `db.col.find({"likes":{$lte:50}}).pretty()` | `where likes <= 50` |
| 大于    | `{:{$gt:}}`  | `db.col.find({"likes":{$gt:50}}).pretty()` | `where likes > 50`  |
| 大于或等于 | `{:{$gte:}}` | `db.col.find({"likes":{$gte:50}}).pretty()` | `where likes >= 50` |
| 不等于   | `{:{$ne:}}`  | `db.col.find({"likes":{$ne:50}}).pretty()` | `where likes != 50` |

## and查询

当需要以多个键值对为条件查询时，默认使用and关系查询

```json
> db.user.find({uname:'gaofeng',age:15}).pretty()
{
        "_id" : ObjectId("5c8b72bf4a5c57c4ada08ab8"),
        "uname" : "gaofeng",
        "age" : 15
}

//类似于SQL中的：where uname = 'gaofeng' and age = 15
```

## or查询

使用$or关键字，后面跟条件数组

```json
> db.user.find({$or:[{uname:'zhangsan'},{age:15}]}).pretty()
{
        "_id" : ObjectId("5c8b72bf4a5c57c4ada08ab8"),
        "uname" : "gaofeng",
        "age" : 15
}
{
        "_id" : ObjectId("5c8f66974a5c57c4ada08aba"),
        "uname" : "zhangsan",
        "age" : 25,
        "address" : "beijing"
}
//类似于SQL中的： where uname = 'zhangsan' or age = 15
```

## and 和 or 联合查询

```json
> db.user.find({age:{$gt:15},$or:[{uname:'zhangsan'},{address:'beijing'}]}).pretty()
{
        "_id" : ObjectId("5c8f66974a5c57c4ada08aba"),
        "uname" : "zhangsan",
        "age" : 25,
        "address" : "beijing"
}
// where age > 15 and (uname = 'zhangsan' or address = 'beijing')
```

## $type 查询操作符

在mongoDB的集合中，一个键可以对应多种类型的数据，例如：uname可以是代号12345，也可以是字符串`gaofeng`。那么我们想查询uname的值为字符串的文档，怎么查询呢，就使用到了$type

```json
> db.user.update({uname:'zhangsan'},{$set:{age:'25'}})   //先把名字为zhangsan的年龄改为字符串
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.user.find({age:{$type:'string'}}).pretty()  //查询年龄值为字符串类型的所有文档
{
        "_id" : ObjectId("5c8f66974a5c57c4ada08aba"),
        "uname" : "zhangsan",
        "age" : "25",
        "address" : "beijing"
}
```

## limit() 和 skip()实现分页查询

db.col.find().limit(NUMBER)，其中NUMBER参数表示要查询多少条数据

db.col.find().limit(NUMBER).skip(COUNT)，skip()方法表示跳过多少条数据

```json
> db.user.find()   //查询所有
{ "_id" : ObjectId("5c8b716e4a5c57c4ada08ab6"), "uname" : "gaofeng", "age" : 28, "address" : "shanghai" }
{ "_id" : ObjectId("5c8b72a74a5c57c4ada08ab7") }
{ "_id" : ObjectId("5c8b72bf4a5c57c4ada08ab8"), "uname" : "gaofeng", "age" : 15 }
{ "_id" : ObjectId("5c8effca4a5c57c4ada08ab9"), "user" : "xiaoming" }
{ "_id" : ObjectId("5c8f66974a5c57c4ada08aba"), "uname" : "zhangsan", "age" : "25", "address" : "beijing" }
> db.user.find().limit(2) //查询两条数据
{ "_id" : ObjectId("5c8b716e4a5c57c4ada08ab6"), "uname" : "gaofeng", "age" : 28, "address" : "shanghai" }
{ "_id" : ObjectId("5c8b72a74a5c57c4ada08ab7") }
//假如每页2条数据，我想查询第二页的数据，也就是当前页-1*2，这是要跳过的条数
> db.user.find().limit(2).skip(2)
{ "_id" : ObjectId("5c8b72bf4a5c57c4ada08ab8"), "uname" : "gaofeng", "age" : 15 }
{ "_id" : ObjectId("5c8effca4a5c57c4ada08ab9"), "user" : "xiaoming" }
>

```

## sort()排序

1 表示升序  ，-1 表示降序

```json
> db.user.update({age:{$type:'string'}},{$set:{age:25}})  //先把年龄改成数字
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.user.find().sort({age:1})  //按照年龄升序
{ "_id" : ObjectId("5c8b72a74a5c57c4ada08ab7") }
{ "_id" : ObjectId("5c8effca4a5c57c4ada08ab9"), "user" : "xiaoming" }
{ "_id" : ObjectId("5c8b72bf4a5c57c4ada08ab8"), "uname" : "gaofeng", "age" : 15 }
{ "_id" : ObjectId("5c8f66974a5c57c4ada08aba"), "uname" : "zhangsan", "age" : 25, "address" : "beijing" }
{ "_id" : ObjectId("5c8b716e4a5c57c4ada08ab6"), "uname" : "gaofeng", "age" : 28, "address" : "shanghai" }
```

