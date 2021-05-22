---
title: "Vue Study"
date: 2021-04-28T22:06:57+08:00
lastmod: 2021-04-28T22:06:57+08:00
draft: true
keywords: ["vue"]
description: "vue study"
tags: ["vue"]
categories: ["vue"]
author: "Kyle Zhao"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: false
autoCollapseToc: false
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: true
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""

---

Vue 学习笔记

<!--more-->

vue 是一套用于构建用户界面的渐进式框架。

```html
<div id="app">
  {{ message }}
</div>
```

```js
var app = new Vue({
  // element 缩写
  el: '#app',
  data: {
    message: 'Hello Vue!'
  }
})
```

Vue 将其挂在到 DOM 元素上，然后对其进行控制,



## 指令

指令的职责是：当表达式的值改变时，将其产生的连带影响，响应式的作用于 DOM

Vue 提供了一个强大的过渡效果系统，可以在Vue 插入/更新/移除元素时自动应用过渡效果。

`v-bind`

将这个元素的 `title` 属性和Vue 实例的 message property 保持一致。

```html
<div id="app-2">
  <span v-bind:title="message">
    鼠标悬停几秒钟查看此处动态绑定的提示信息！
  </span>
  <!-- 缩写 -->
  <span :title="message">
    鼠标悬停几秒钟查看此处动态绑定的提示信息！
  </span>
</div>

```

使用表达式，模板表达式都被放在沙盒中，只能访问 全局变量的一个白名单，如 `Math` 和 `Date`, 不应该在模板表达式中试图访问用户定义的全局变量。

```html
<div v-bind:id="'list-' + id"></div>
```

2.6.0 动态参数

```html
<a v-bind:[attributeName]="url"> ... </a>
<a v-on:[eventName]="doSomething"> ... </a>
<!--
避免使用大写字符来命名键名
在 DOM 中使用模板时这段代码会被转换为 `v-bind:[someattr]`。
除非在实例中有一个名为“someattr”的 property，否则代码不会工作。
-->
<a v-bind:[someAttr]="value"> ... </a>
```

`v-if` `v-else`

只在指令的表达式返回 true时候被渲染， 不仅可以把数据绑定到 DOM 文本或attribute，还可以绑定到 DOM 结构。

v-if 是完全移除，v-show 是显示隐藏。

```
<div id="app-3">
  <p v-if="seen">现在你看到我了</p>
  <p v-else>我让你看不见</p>
</div>
```

`v-for`

```html
<div id="app-4">
  <ol>
    <li v-for="todo in todos">
      {{ todo.text }}
    </li>
  </ol>
</div>
```

```js
var app4 = new Vue({
  el: '#app-4',
  data: {
    todos: [
      { text: '学习 JavaScript' },
      { text: '学习 Vue' },
      { text: '整个牛项目' }
    ]
  }
})
```

在控制台里，输入 `app4.todos.push({ text: '新项目' })`，你会发现列表最后添加了一个新项目。

`v-on`

添加事件监听器，通过它调用在 Vue 实例中定义的方法。

```html
<div id="app-5">
  <p>{{ message }}</p>
  <button v-on:click="reverseMessage">反转消息</button>
  <button @click="reverseMessage">反转消息</button>
    <!-- 动态参数的缩写 (2.6.0+) -->
  <a @[event]="doSomething"> ... </a>
</div>
```

```js
var app5 = new Vue({
  el: '#app-5',
  data: {
    message: 'Hello Vue.js!'
  },
  methods: {
    reverseMessage: function () {
      this.message = this.message.split('').reverse().join('')
    }
  }
})
```

修饰符是以半角句号 `.`指明的特殊后缀，用以指出一个指令应该以特殊方式绑定，例如， .prevent 修饰符告诉 v-on 指令对于触发的事件调用 event.preventDefault()

```html
<form v-on:submit.prevent="onSubmit">...</form>
```



`v-model`

轻松实现表单输入和应用状态之间的双向绑定

```html
<div id="app-6">
  <p>{{ message }}</p>
  <input v-model="message">
</div>
```

```js
var app6 = new Vue({
  el: '#app-6',
  data: {
    message: 'Hello Vue!'
  }
})
```

`v-once`

执行一次性地插值，当数据改变时，插值处的内容不会更新，但请留心这会影响到该节点上的其他数据绑定

```html
<span v-once>这个将不会改变：{{ msg }}</span>
```

`v-html`

双大括号会将数据解释为普通文本，而非 HTML 代码，为了输出真正的 THML ，你需要使用 v-html 指令

```html
<p>Using mustaches: {{ rawHtml }}</p>
<p>Using v-html directive: <span v-html="rawHtml"></span>
```

> 站点上动态渲染的任意HTML可能会非常危险，因为它容易导致 XSS 攻击，请只对可信内容使用 HTML 插值，绝不要 对用户提供的内容使用插值。

## 组件化应用构建

几乎任意类型的应用界面都可以抽象为一个组件树，在 Vue里，一个组件本质上是一个拥有预定义选项的一个 Vue实例。

```html
<div id="app-7">
  <ol>
    <!--
      现在我们为每个 todo-item 提供 todo 对象
      todo 对象是变量，即其内容可以是动态的。
      我们也需要为每个组件提供一个“key”，稍后再
      作详细解释。
    -->
    <todo-item
      v-for="item in groceryList"
      v-bind:todo="item"
      v-bind:key="item.id"
    ></todo-item>
  </ol>
</div>
```

```js
// 定义名为 todo-item 的组件
Vue.component('todo-item', {
  // todo-item 组件现在接受一个"prop", 类似于一个自定义 attribute。
  props: ['todo'],
  template: '<li>{{ todo.text }}</li>'
})
var app7 = new Vue({
  el: '#app-7',
  data: {
    groceryList: [
      { id: 0, text: '蔬菜' },
      { id: 1, text: '奶酪' },
      { id: 2, text: '随便其它什么人吃的东西' }
    ]
  }
})
```

## 数据与方法

当一个 Vue 实例被创建时，它将 `data` 对象中的所有的 property 加入到 Vue 的响应式系统中。当 这些 property 的值发生改变时，视图将会产生响应，即匹配更新为新的值。 只有当实例被创建时就已经存在于 `data` 中的property 才是`响应式`的, 也就是说如果添加一个新的 property 例如： vm.b = 'hi', 是不会触发任何视图的更新。

除了数据 property，Vue 实例还暴露了一些有用的实例 property 与方法。它们都有前缀 `$`，以便与用户定义的 property 区分开来。例如：

```js
var data = { a: 1 }
var vm = new Vue({
  el: '#example',
  data: data
})

vm.$data === data // => true
vm.$el === document.getElementById('example') // => true

// $watch 是一个实例方法
vm.$watch('a', function (newValue, oldValue) {
  // 这个回调将在 `vm.a` 改变后调用
})
```

## 实例生命周期钩子

`created`

用来在一个实例被创建之后执行代码

```js
new Vue({
  data: {
    a: 1
  },
  created: function () {
    // `this` 指向 vm 实例
    console.log('a is: ' + this.a)
  }
})
```

> 不要在选项 property 或回调上使用[箭头函数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/Arrow_functions)，比如 `created: () => console.log(this.a)` 或 `vm.$watch('a', newValue => this.myMethod())`。因为箭头函数并没有 `this`，`this` 会作为变量一直向上级词法作用域查找，直至找到为止，经常导致 `Uncaught TypeError: Cannot read property of undefined` 或 `Uncaught TypeError: this.myMethod is not a function` 之类的错误。

`mounted`

`updated`

`destoryed`



## 计算属性

```html
<div id="example">
  <p>Original message: "{{ message }}"</p>
  <p>Computed reversed message: "{{ reversedMessage }}"</p>
</div>
```

```js
var vm = new Vue({
  el: '#example',
  data: {
    message: 'Hello'
  },
  computed: {
    // 计算属性的 getter
    reversedMessage: function () {
      // `this` 指向 vm 实例
      return this.message.split('').reverse().join('')
    }
  }
})
```

声明一个计算属性 `reversedMessage`, 提供的函数将用作 property `vm.reversedMessage` 的 getter 函数:

```js
console.log(vm.reversedMessage) // => 'olleH'
vm.message = 'Goodbye'
console.log(vm.reversedMessage) // => 'eybdooG
```

当 vm.message 改变时,所有依赖 vm.reversedMessage 的绑定也会更新。

## 计算属性缓存 vs 方法

```html
<p>Reversed message: "{{ reversedMessage() }}"</p>
```

```
// 在组件中
methods: {
  reversedMessage: function () {
    return this.message.split('').reverse().join('')
  }
}
```

两种方式的最终结果是相同的，不同的是计算属性是基于它们的响应式依赖进行缓存的，只在相关响应式依赖发生改变时它们才会重新求值，这就意味着只要 message 还没有发生改变，多次访问 reversedMessage 计算属性会立即返回之前的计算结果，而不必再次执行函数。

下面的计算属性将不再更新，因为 Date.now() 不是响应式依赖

```js
computed: {
  now: function () {
    return Date.now()
  }
}
```

相比之下，每当触发重新渲染时，调用方法总会再次执行函数。

## 计算属性 vs 侦听属性

Vue 提供了一种更通用的方式来观察和响应 Vue 实例上的数据变动：侦听属性。

当你有一些数据需要随着其他数据变动而变动时，你很容易滥用 watch，通常更好的做法是使用计算属性而不是命令式的 watch 回调。

```html
<div id="demo">{{ fullName }}</div>
```

```js
var vm = new Vue({
  el: '#demo',
  data: {
    firstName: 'Foo',
    lastName: 'Bar',
    fullName: 'Foo Bar'
  },
  watch: {
    firstName: function (val) {
      this.fullName = val + ' ' + this.lastName
    },
    lastName: function (val) {
      this.fullName = this.firstName + ' ' + val
    }
  }
})
```

上面代码是命令式且重复的。将它与计算属性的版本进行比较：

```js
var vm = new Vue({
  el: '#demo',
  data: {
    firstName: 'Foo',
    lastName: 'Bar'
  },
  computed: {
    fullName: function () {
      return this.firstName + ' ' + this.lastName
    }
  }
})
```

## 计算属性的 setter

计算属性默认只有 getter，不过在需要时也可以提供一个 setter：

```js
computed: {
  fullName: {
    // getter
    get: function () {
      return this.firstName + ' ' + this.lastName
    },
    // setter
    set: function (newValue) {
      var names = newValue.split(' ')
      this.firstName = names[0]
      this.lastName = names[names.length - 1]
    }
  }
}
```

运行 vm.fullNaame = 'John Doe'时，setter 会被调用， vm.firstName 和 vm.lastName 也会相应地被更新。

## 侦听器

虽然计算属性在大多数情况下更合适，但有时也需要一个自定义的侦听器。

当需要在数据变化时执行异步或开销较大的操作时，这个方式是最有用的。

```html
<div id="watch-example">
  <p>
    Ask a yes/no question:
    <input v-model="question">
  </p>
  <p>{{ answer }}</p>
</div>
```

```js
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
```

在这个示例中，使用 `watch` 选项允许我们执行异步操作 (访问一个 API)，限制我们执行该操作的频率，并在我们得到最终结果前，设置中间状态。这些都是计算属性无法做到的。

除了 watch 之外，还可以使用命令式的 vm.$watch API.

