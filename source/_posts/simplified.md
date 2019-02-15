---
title: 偶然的收获java实体类简化
date: 2017-07-27 14:23:15
tags: java
---

今天编写实体bean的时候，正好一位大拿从身后飘过，说：来，我教你一种可以不写set，get的方法。当时还在想，还有这么好的事。

    原来，彪哥用了Lombok，其实是一个开源的jar包，引入包，使用包中提供的注解，就可以不用写set，get方法了。其实是javac的一个插件，当我们写好的代码在编译的过程中，javac会扫描带有该注解的实体类，并自动生成set，get方法到class文件中，因此和你自己写的set，get方法一样，也不会影响性能，因为他是在编译阶段完成的。

    网上也有很多介绍Lombok的内容，我认为自己也整理一套吧。

 <!-- more -->

  ** 下载安装 **

    lombok 的官方网址：[http://projectlombok.org/](http://projectlombok.org/)  可以下载jar包

    1. 双击下载下来的 JAR 包安装 lombok
           我选择这种方式安装的时候提示没有发现任何 IDE，所以我没安装成功，我是手动安装的。如果你想以这种方式安装，请参考官网的视频。
       　2.eclipse / myeclipse 手动安装 lombok
    2. 将 lombok.jar 复制到 myeclipse.ini / eclipse.ini 所在的文件夹目录下
    3. 打开 eclipse.ini / myeclipse.ini，在最后面插入以下两行并保存：
               -Xbootclasspath/a:lombok.jar
               -javaagent:lombok.jar
           3.重启 eclipse / myeclipse

   以上步骤我都做完之后，依然提示找不到@Getter注解，我试着把jar包直接添加到项目的lib目录下，成功了。

  

    **测试**

```
package com.entry;

import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class Person {

    private int id;
    private String name;

    public Person() {

    }

    public Person(int id, String name) {
        this.id = id;
        this.name = name;
    }

    @Override
    public String toString() {
        return "编号：" + id + " 姓名：" + name;
    }

    public static void main(String[] args) {
        Person per = new Person();
        per.setId(123);
        per.setName("张三疯");
        System.out.println(per.toString());// 编号：123 姓名：张三疯
        
        Person per2 = new Person(124, "李二蛋");
        System.out.println("编号："+per2.getId()+" 姓名："+per2.getName());//编号：124 姓名：李二蛋
    }

}
```

　　想想，如果这个类有那么50，60个属性的话，得有多少个set，get方法。

 

当然一些特殊情况，我们需要在调用set或者get方法的时候，需要做特殊处理的时候，我们也可以自己编写set，get方法，这个时候Lombok便不再为该属性添加set或者get方法

```
package com.entry;

import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class Person {

    private int id;
    private String name;

    // 自己写了，lombok将不再生成该方法
    public void setName(String name) {
        this.name = name + "你好!";
    }

    public int getId() {
        return id + 10000;
    }

    public Person() {

    }

    public Person(int id, String name) {
        this.id = id;
        this.name = name;
    }

    @Override
    public String toString() {
        return "编号：" + id + " 姓名：" + name;
    }

    public static void main(String[] args) {
        Person per = new Person();
        per.setId(123);
        per.setName("张三疯");
        System.out.println(per.toString());// 编号：123 姓名：张三疯你好!

        Person per2 = new Person(124, "李二蛋");
        System.out.println("编号：" + per2.getId() + " 姓名：" + per2.getName());//编号：10124 姓名：李二蛋
    }

}
```

 

 

    **lombok 注解**：
    lombok 提供的注解不多，可以参考官方视频的讲解和官方文档。
    Lombok 注解在线帮助文档：[http://projectlombok.org/features/index.](http://projectlombok.org/features/index.html)    下面介绍几个我常用的 lombok 注解：
        @Data   ：注解在类上；提供类所有属性的 getting 和 setting 方法，此外还提供了equals、canEqual、hashCode、toString 方法
        @Setter：注解在属性上；为属性提供 setting 方法
        @Getter：注解在属性上；为属性提供 getting 方法
        @Log4j ：注解在类上；为类提供一个 属性名为log 的 log4j 日志对象
        @NoArgsConstructor：注解在类上；为类提供一个无参的构造方法
        @AllArgsConstructor：注解在类上；为类提供一个全参的构造方法