---
title: Vue中的计算属性和侦听器
date: 2018-11-06 09:36:48
tags: [vue]
---

# 计算属性

当我们要显示一个需要依赖现有属性，通过逻辑运算得到的结果时，vue引入了计算属性的概念。计算属性的值依赖于现有属性，只有当现有属性发生改变的时候，才会重新计算计算属性的值（缓存）。所以，对于任何复杂的逻辑，都应当使用计算属性。

<!-- more -->

# 声明计算属性

```js
        var vm = new Vue({
          el:'#my',
          data:{
            message:'hello vue.js'
          },
          computed:{//声明计算属性
            reverseMessage:function(){//这里的function是作用在reverseMessage的getter方法上
              return this.message.split(' ').reverse().join(' ');
            }
          }
        });
```

```html
    <div id="my">
        {{message}}
        <hr>
        {{reverseMessage}}<!-- 显示计算属性和显示普通属性一样 -->
    </div>
```

>    hello vue.js
>
>    vue.js hello

## 计算属性VS方法

```js
      var vm = new Vue({
          el:'#my',
          data:{
            message:'hello vue.js'
          },
          computed:{
            reverseMessage:function(){
              return this.message.split(' ').reverse().join(' ');
            }
          },
          methods:{//声明方法
            rmessage:function(){
              return this.message.split(' ').reverse().join(' ');
            }
          }
        });
```

```html
    <div id="my">
        {{message}}
        <hr>
        {{reverseMessage}}
        <hr>
        {{rmessage()}}
    </div>
```

>    hello vue.js
>
>    vue.js hello
>
>    vue.js hello

我们可以看到，效果是一样的，那他们有什么区别呢，想象一下，如果这个结果需要复杂的逻辑运算，消耗大量的资源的话，没调用一次方法，都会消耗一次。而计算属性呢，只计算一次，只有当依赖属性变化的时候才会再次计算，从性能上来讲，肯定是计算属性要高。当然如果不需要使用缓存的话，可以使用方法。

# 侦听器

侦听器的效果和计算属性类是，当需要在数据变化时执行异步或开销较大的操作时可以使用侦听器，例如查询功能。使用 `watch` 选项允许我们执行异步操作 (访问一个 API)，限制我们执行该操作的频率，并在我们得到最终结果前，设置中间状态。这些都是计算属性无法做到的。

```html
<!DOCTYPE html>
<html lang="en" dir="ltr">

<head>
  <meta charset="utf-8">
  <title></title>
  <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
</head>

<body>
  <div id="watch-example">
    <p>
      Ask a yes/no question:
      <input v-model="question">
    </p>
    <p>{{ answer }}</p>
  </div>
  <!-- 因为 AJAX 库和通用工具的生态已经相当丰富，Vue 核心代码没有重复 -->
  <!-- 提供这些功能以保持精简。这也可以让你自由选择自己更熟悉的工具。 -->
  <script src="https://cdn.jsdelivr.net/npm/axios@0.12.0/dist/axios.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/lodash@4.13.1/lodash.min.js"></script>
  <script>
  var watchExampleVM = new Vue({
    el: '#watch-example',
    data: {
      question: '',
      answer: 'I cannot give you an answer until you ask a question!'
    },
    watch: {
      // 如果 `question` 发生改变，这个函数就会运行
      question: function (newQuestion, oldQuestion) {
        this.answer = 'Waiting for you to stop typing...'
        this.debouncedGetAnswer()
      }
    },
    created: function () {
      // `_.debounce` 是一个通过 Lodash 限制操作频率的函数。
      // 在这个例子中，我们希望限制访问 yesno.wtf/api 的频率
      // AJAX 请求直到用户输入完毕才会发出。想要了解更多关于
      // `_.debounce` 函数 (及其近亲 `_.throttle`) 的知识，
      // 请参考：https://lodash.com/docs#debounce
      this.debouncedGetAnswer = _.debounce(this.getAnswer, 500)
    },
    methods: {
      getAnswer: function () {
        if (this.question.indexOf('?') === -1) {
          this.answer = 'Questions usually contain a question mark. ;-)'
          return
        }
        this.answer = 'Thinking...'
        var vm = this
        axios.get('https://yesno.wtf/api')
          .then(function (response) {
            vm.answer = _.capitalize(response.data.answer)
          })
          .catch(function (error) {
            vm.answer = 'Error! Could not reach the API. ' + error
          })
      }
    }
  })
  </script>
</body>

</html>

```

