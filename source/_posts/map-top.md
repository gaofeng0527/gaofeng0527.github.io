---
title: map-top
date: 2018-07-10 09:07:12
tags: [test]
---

# 编写类对象

板块类

```java
package example.spring.beans;

/**
 * 板块类
 */
public class Kind {

    private int kindId;//板块id
    private String kindName;//板块名称

    public Kind(){ }
    public Kind(int kindId, String kindName) {
        this.kindId = kindId;
        this.kindName = kindName;
    }
    public int getKindId() {
        return kindId;
    }

    public void setKindId(int kindId) {
        this.kindId = kindId;
    }

    public String getKindName() {
        return kindName;
    }

    public void setKindName(String kindName) {
        this.kindName = kindName;
    }
}
```

<!-- more -->

主贴类

```java
package example.spring.beans;

/**
 * 主贴类
 */
public class Topic {

    private int topId;//主贴id
    private String title;//主贴标题
    private int kindId;//板块id

    public int getTopId() {
        return topId;
    }
    public void setTopId(int topId) {
        this.topId = topId;
    }
    public String getTitle() {
        return title;
    }
    public void setTitle(String title) {
        this.title = title;
    }
    public int getKindId() {
        return kindId;
    }
    public void setKindId(int kindId) {
        this.kindId = kindId;
    }

    public Topic() {
    }

    public Topic(int topId, String title, int kindId) {
        this.topId = topId;
        this.title = title;
        this.kindId = kindId;
    }
}

```

# 编写service

```java
package example.spring.service;

import example.spring.beans.Kind;
import example.spring.beans.Topic;

import java.util.*;

public class TopicService {


    /**
     * 获取所有板块，以及板块下的所有主贴
     * 数据我就模拟了，不连接数据库了
     * @return
     */
    public Map<String,List<Topic>> findAllTopic(){
        //1.获取所有板块
        List<Kind> klist = this.findAllKind();

        //2.声明一个map用来存储最终结果集
        Map<String,List<Topic>> map = new HashMap<>();

        //3.循环所有板块，根据板块Id获取所有该板块下的主贴
        for(Kind kind :klist){
            // 根据板块的Id查询主贴
            List<Topic> tlist = this.findTopicByKindId(kind.getKindId());
            //已板块名称作为key，主贴集合作为value
            map.put(kind.getKindName(),tlist);
        }
        return map;
    }

    /**
     * 模拟主贴表
     * @param kindId
     * @return
     */
    public List<Topic> findTopicByKindId(int kindId){
        List<Topic> tlist = new ArrayList<>();
        if(kindId == 1){
            tlist.add(new Topic(1,"java开发基础",1));
            tlist.add(new Topic(2,"java.IO使用手册",1));
            tlist.add(new Topic(3,"javaWEB开发",1));
        }else if(kindId == 2){
            tlist.add(new Topic(4,".net开发基础",2));
            tlist.add(new Topic(5,"ASP使用手册",2));
        }else if(kindId == 3){
            tlist.add(new Topic(6,"android sdk 帮助手册",3));
            tlist.add(new Topic(7,"android适配宝典",3));
            tlist.add(new Topic(8,"android监听器的使用",3));
        }

        return tlist;
    }


    /**
     * 获取所有板块
     * @return
     */
    public List<Kind> findAllKind(){
        List<Kind> klist = new ArrayList<>();
        klist.add(new Kind(1,"java开发"));
        klist.add(new Kind(2,".net开发"));
        klist.add(new Kind(3,"android开发"));
        return klist;
    }


    /**
     * 测试
     * @param args
     */
    public static void main(String[] args) {
        TopicService ts = new TopicService();
        Map<String,List<Topic>> map = ts.findAllTopic();
        Set<String> sets = map.keySet();
        for(String str : sets){
            List<Topic> tlist = map.get(str);
            System.out.println(str+"------------------");
            for (Topic top : tlist){
                System.out.println("    "+top.getTopId()+"    "+top.getTitle()+"    ");
            }
            System.out.println("==============================================");
        }
    }
}

```

# 测试结果

>    android开发------------------
>
>        6    android sdk 帮助手册    
>        7    android适配宝典    
>        8    android监听器的使用    
>    ==============================================
>    java开发------------------
>        1    java开发基础    
>        2    java.IO使用手册    
>        3    javaWEB开发    
>    ==============================================
>    .net开发------------------
>        4    .net开发基础    
>        5    ASP使用手册    
>    ==============================================