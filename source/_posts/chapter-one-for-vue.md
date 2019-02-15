---
title: 初识VUE
date: 2018-11-05 09:39:47
tags: [vue]
---

# Vue简介

Vue是一套用于构建用户界面的`渐进式框架`,Vue的核心库只关注视图层，Vue通过操控数据来驱动视图层的变化。学习成本不高，下面开始学习Vue吧。

Vue可以通过页面引入js文件来使用，也可以通过 vue cli创建Vue项目工程，初学者建议先用简单的页面引用js文件，来慢慢熟悉Vue。使用Vue时，要放弃自己操作dom的想法，因为vue会帮你做。

<!-- more -->

# 引入Vue.js

引入Vue.js有多种方式

>    1.官网下载js文件，存放到本地，在页面中引入
>
>    2.直接在页面中引入这段代码
>
>    ```html
>    <!-- 开发环境版本，包含了有帮助的命令行警告 -->
>    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"/>
>    <!-- 开发环境版本，包含了有帮助的命令行警告 -->
>    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"/>
>    ```
>

# 第一Vue实例

```html
<!DOCTYPE html>
<html lang="en" dir="ltr">
  <head>
    <meta charset="utf-8">
    <title></title>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
  </head>
  <body>

    <script type="text/javascript">
      window.onload=function(){
        new Vue({
          el:'#my',//2.0以后版本不允许直接挂载到html,body等元素标签上
          data:{
            msg:'hello word!'
          }
        });
      }
    </script>

    <div id="my">
      {{msg}}
    </div>
  </body>
</html>
```

访问页面输出内容：

>    hello word!

1.new Vue()创建了一个vue实例，el表示element，根据ID选择器选中一个dom模块作为该vue的操作区域，在该区域中可以调用vue中的数据，方法等。例如上面的例子，在vue实例中，在data里声明msg数据，在下方Id为my的div中，使用{{msg}}显示vue中定义的内容。

# Vue中的指令

指令带有前缀：v-，以表示它们是Vue提供的特殊特性，其实就是自定义属性。它们会在渲染的dom上应用特殊的响应式行为。

`v-bind`  绑定指令，例如：将这个元素节点的title特性和vue中的message属性保持一致，谁的值变化，两个都会同步。

```html
<scan v-bind:title="message">鼠标悬浮一会，会看到弹出的title</scan>
```

`v-if`、`v-else-if`、`v-else` Vue中的条件判断

```html
<div id="app1">
	<p v-if="day==1">今天星期一</p>
    <p v-else-if="day == 2">今天星期二</p>
    <p v-else>今天星期三</p>
</div>

```

```js
new Vue({
  el:"#app1",
  data:{
    day:3
  }
});
```

`v-for` 循环指令

```js
new Vue({
   el:"#app3",
   data:{
        courses:[
         '数学','英语','语文'
        ] //vue中声明数组
       }
 });
```



```html
<div id="app3">
   <ul>
      <li v-for="course in courses">{{course}}</li>
   </ul>
</div>
```

>    -    数学
>    -    英语
>    -    语文

也可以使用索引形式

```html
	<div id="app3">
      <ul>
          <li v-for="(course,index) in courses">{{index}}.{{course}}</li>
      </ul>
    </div>
    <!-- 这里的index表示数组索引 -->
```

在Vue中对象数组也用v-for指令来循环

`v-on` 该指令可以给dom上添加事件，例如：click，load，unload，change，mouseover，mouseout事件等等。

```js
new Vue({
	el:"#app4",
    data:{
    	message:'hello Vue.js'
    },
    methods:{//methods属性中声明vue方法
    	reverseMessage:function(){
        	this.message = this.message.split(' ').reverse().join(' ');
        }
    }
});
```

```html
	<div id="app4">
    	{{message}}
    	<button v-on:click="reverseMessage">逆转消息</button>
    </div>
```

`v-model` 该指令可以实现表单输入和应用状态直接的双向绑定。修改输入框的值，message的值也会跟着变化，同样修改message的值，text输入框也跟着变化

```js
        new Vue({
          el:"#app5",
          data:{
            message:'hello Vue.js'
          }
        });
```

```html
    <div id="app5">
    {{message}}
    <input type="text" v-model="message">
    </div>
```

`v-html` 该指令可以输出html文本内容

在上面的例子中{{message}}可以输出内容，会把原始的文本内容输出，如果想显示html内容的话，可以使用v-html指令进行绑定

```html
    <div id="app6">
      <p>{{message}}</p>
      <p><scan v-html="message"></scan></p>
    </div>
```

```js
		var vue6 = new Vue({
          el:"#app6",
          data:{
            message:'<scan style="color:red">Hello Vue.js</scan>'
          }
        });
```

>    1.<scan style="color:red">Hello Vue.js</scan>
>
>    2.Hello Vue.js(红色)