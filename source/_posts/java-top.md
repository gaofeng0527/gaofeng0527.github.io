---
title: 日常积累
date: 2019-02-15 11:52:32
tags: java
---

# 比较Long类型 20190215

平时比较数字类型时习惯直接用`==`来比较，有一次发现两个Long类型的数字直接用`==`比较没有达到预期效果，仔细一想Long类型是long的封装类型，`==`比较是比较的引用地址，因此值是一样的，却返回false。

```java
Long l1 = new Long(10);
Long l2 = new Long(10);
System.out.println(l1 == l2);//false
System.out.println(l1.longValue() == l2.longValue());//true
```

上面代码也可以看出：long类型是java中的基本数据类型，可以用`==`直接比较

<!-- more -->

# cookie的读取 20190215

之前没有添加这一句`cookie.setPath("/");`导致在读取cookie的时候失败，是因为cookie默认的作用域是在同一路径下。`/write/cookie` 和 `/read/cookie` 不属于同一路径。

```java
@RequestMapping("/write/cookie")
    @ResponseBody
    public String writeCookie(HttpServletResponse response) {
        Cookie cookie = new Cookie("username", "Don't tell me");
        cookie.setMaxAge(10000);
        cookie.setPath("/");
        response.addCookie(cookie);
        return "write cookie username";
    }
```

```java
/**
     * 在添加：cookie.setPath("/");这一行代码之前，读取不到cookie，因为cookie默认的作用域是在同一路径下有效
     * 添加这一行代码后，是在同一域名下有效
     * @param cookie
     * @return
     */
    @RequestMapping("/read/cookie")
    @ResponseBody
    public String readCookie(@CookieValue(value = "username",defaultValue = "") String cookie) {
        return "cookie:username ==> "+cookie;
    }
```

# GitHub保存笔记源文件20190215

1.   在`hexo`下创建笔记 

     >    hexo new "fileName"
     >
     >    hexo generate
     >
     >    hexo deploy 

2.   在Blog目录下执行git命令把修改添加到Git仓库中

     >    git add <file>
     >
     >    git commit -m "message"
     >
     >    git push -u origin master

这样，换到任何环境，都不会丢失自己之前写过的笔记了（除非github不再免费存储了）

# idea 本地Debugger 20190221

1.   在idea操作页面 Run-->Edit-configurations-->点击左上角的+号--->在弹出的页面中选中Remote（开启远程监听）
2.   Name 输入一个名称；setting都默认就好；Host 输入要测试的服务器ID，本地的话默认就好；Port 测试监听的端口，本地默认；Search sources using module's classpath 选中要测试的代码路径；jvm默认：-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005
3.   启动测试的服务
4.   启动远程监听
5.   在需要调试的地方加上断点，即可进行断点调试了 

# JAVA 解析XML

参考[XML解析——Java中XML的四种解析方式](https://www.cnblogs.com/longqingyang/p/5577937.html)

# JAVA 解析 HTML

导入需要的jar

``` gradle
compile('org.jsoup:jsoup:1.11.3')
```

```java
/**
     * 根据HTMl解析题目的内容
     *
     * @param content
     * @return
     */
    private Map<String, Object> getQuestionByHtml(String content) {
        org.jsoup.nodes.Document dom = Jsoup.parse(content);

        //获取最外层信息（题目ID，知识点编码）根据样式属性获取
        Elements elements = dom.select("div.popup-questions");
        String knowID = "";
        String questionID = "";
        if (null != elements && elements.size() > 0) {
            Element element = elements.get(0);
            knowID = element.attr("knowid");//获取属性的值
            questionID = element.attr("id");
        }

        //获取题目配置信息
        Elements qelements = dom.select("div.question");
        String code = "";
        String type = "";
        String answer = "";
        String score = "";
        if (null != qelements && qelements.size() > 0) {
            Element qelement = qelements.get(0);
            code = qelement.attr("code");
            type = qelement.attr("type");
            answer = qelement.attr("answer");
            score = qelement.attr("score");
        }

        //获取题干
        Elements titles = dom.select("p.label");
        String title = "";
        if (null != titles && titles.size() > 0) {
            title = titles.get(0).html();
        }

        //答案解析
        String analysis = "";
        Elements eanalysis = dom.select("p.label");
        if (null != eanalysis && eanalysis.size() > 0) {
            analysis = eanalysis.get(0).html();
        }

        //解析选项
        Elements options = dom.getElementsByTag("li");
        Map<String, String> optionMap = new HashMap<>();
        for (Element option : options) {
            optionMap.put(option.attr("code"), option.html());
        }
        //组装
        Map<String, Object> result = new HashMap<>();
        Map<String, Object> question = new HashMap<>();
        question.put("id", questionID);
        question.put("code", code);
        question.put("type", type);
        question.put("answer", answer);
        question.put("score", score);
        question.put("title", title);
        question.put("knowid", knowID);
        question.put("options", optionMap);
        question.put("analysis", analysis);
        result.put(knowID, question);
        return result;
    }
```

# JAVA处理HTML时，去掉标签内部的属性20190315

```java
	private static final String regEx_tag = "<(\\w[^>|\\s]*)[\\s\\S]*?>";

    public static String removeEleProp(String htmlStr) {
        Pattern p = Pattern.compile(regEx_tag, Pattern.CASE_INSENSITIVE);
        Matcher m = p.matcher(htmlStr);
        StringBuffer sb = new StringBuffer();
        while (m.find()) {
            String tagWithProp= m.group(0);
            String tag =m.group(1);
            if ("img".equals(tag)) {
                //img标签保留属性，可进一步处理删除无用属性，仅保留src等必要属性
                m.appendReplacement(sb, tagWithProp);
            }else if ("a".equals(tag)) {
                //a标签保留属性，可进一步处理删除无用属性，仅保留href等必要属性
                m.appendReplacement(sb, tagWithProp);
            }else{
                m.appendReplacement(sb, "<" + tag + ">");
            }
        }
        m.appendTail(sb);
        return sb.toString();
    }
```

# eclipse+tomcat远程调试20190319

tomcat 默认提供8000为测试端口

使用命令：./catalina.sh jpda start   启动服务

在eclipse中创建远程监听，就可以进行远程断点调试了。

# List遍历过程中操作List中的元素20190627

如下代码：

```java
List<String> names = new ArrayList<>();
names.add("张三");
names.add("李四");
names.add("王五");
names.add("张二麻子");
/**
* 1.抛出java.util.ConcurrentModificationException
* 是因为在移除list中的元素时，list的长度也跟着变化，因此继续访问的话，就会数组越界了
*
*/
for (String name : names) {
	if("李四".equals(name)){
    	names.remove(name);
    }
}
```

正确做法：

```java
		Iterator<String> is = names.iterator();
        while (is.hasNext()){
            if("李四".equals(is.next())){
                is.remove();
            }
        }
```

另外`ListIterator`对象提供了`hasPrevious()`方法可以判断前面是否还有元素。因此用这个迭代器可以逆向遍历集合。

# List快速去重20190627

大家都知道List是有序的可重复的单列列表集合。那么如何快速去重呢。想一想set的集合的特性，无序的不可重复的单项列表集合。

>    集合的有序无序指的是，这里的有序和无序不是指集合中的排序，而是是否按照元素添加的顺序来存储对象。

```java
        List<String> list = new ArrayList<>();
        list.add("a");
        list.add("e");
        list.add("c");
        list.add("d");
        list.add("c");
        list.add("a");
        Set<String> set = new TreeSet<>();
        set.addAll(list);
        System.out.println(set);//[a, c, d, e]
```

>    1.   `ArrayList`集合查询元素效率较高
>    2.   `LinkedList`因为是连表形式的数据结构，因此添加，修改，删除效率比较高。
>    3.   `ArrayList`、`LinkedList`都是非线程安全的；`Vector`线程安全的

# Set判断是否重复的原理

在`Set`集合掉用`add`存储元素的时候，会判断`hash`值和`equals`都相同的情况下，才算是重复元素。因此如果存储自定义引用类型的元素时，要确保不重复的话，需要重写`hashCode`和`equals`方法。

`HashSet` 无序不可重复的集合。`LinkedHashSet`是`HashSet`的子类，区别是，`LinkedHashSet`是有序的

# TreeSet

`TreeSet`集合内部的值是自动排序的。如果自定义引用类型，想进行排序的话，需要实现`Comparble`接口并实现`comparTo`方法。

`TreeSet`集合的元素必须实现`Comparble`接口

`TreeSet`不允许存储`null`值

不是线程安全的