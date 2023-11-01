---
title: "Vue3"
date: 2023-11-01T08:59:21+08:00
lastmod: 2023-11-01T08:59:21+08:00
draft: true
keywords: ["vue3", "vue"]
description: "vue3 study"
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

渐进式框架

- 无需构建步骤，渐进式增强静态的 HTML
- 在任何页面中作为 Web Components 嵌入
- 单页应用 SPA
- 全栈/服务端渲染 SSR
- Jamstack / 静态站点生成 SSG
- 开发桌面端、移动端、WebGL, 甚至是命令行终端中的界面

<!--more-->

# API 风格

选项式 API 和 组合式 API。

当不需要构建工具或在低复杂度的场景中使用 vue，推荐选项式 API

当用 Vue 构建完整的单页应用，推荐采用组合式API + 单文件组件

## 选项式 API

选项式 API 是在  组合式API 基础上实现的

```javascript
<script>
    export default {
		data() {
            return {
                count: 0
            }
        },
        methods: {
            increment() {
                this.count++
            }
        },
        mounted() {
            console.log(`init ${this.count}`)
        }
	}
  </script>
<template>
      <button @click="increment">Count is: {{ count }}</button>
</template>
```

## 组合式 API

```javascript
<script setup>
    import {ref, onMounted } from 'vue'
    // 响应式状态
	const count = ref(0)
    // 修改状态、触发更新的函数
    function increment() {
        count.value++
    }
	// 生命周期钩子
	onMounted(() => {
        console.log(`init ${this.count}`)
    })
</script>
<template>
      <button @click="increment">Count is: {{ count }}</button>
</template>
        
```

# 指令

### `v-bind` 

响应式地绑定一个 attribute

```html
<div v-bind:id="dynamicId"></div>
// 简写
<div :id="dynamicId"></div>
```

动态绑定多个值

```html
const obj = {
	id: "kyle",
	class: "wrapper"
}

<div v-bind="obj"></div>
```

使用表达式

```
{{ number + 1 }}

{{ ok ? 'YES' : 'NO' }}

{{ message.split('').reverse().join('') }}

<div :id="`list-${id}`"></div>
```

调用函数, 每次更新时都会被重新调用

```
<time :title="toTitleDate(date)" :datetime="date">
  {{ formatDate(date) }}
</time>
```

### `v-on`

```
<a v-on:click="doSomething"> ... </a>

<!-- 简写 -->
<a @click="doSomething"> ... </a>
```

动态参数

```
<a v-bind:[attributeName]="url"> ... </a>

<!-- 简写 -->
<a :[attributeName]="url"> ... </a>

<!-- 绑定函数 -->
<a v-on:[eventName]="doSomething"> ... </a>

<!-- 简写 -->
<a @[eventName]="doSomething">
```

### 修饰符 Modifiers

`.prevent` 修饰符会告知 `v-on` 指令对触发的事件调用 `event.preventDefault()`

```
<form @submit.prevent="onSubmit">...</form>

<!-- 单击事件将停止传递 -->
<a @click.stop="doThis"></a>

<!-- 提交事件将不再重新加载页面 -->
<form @submit.prevent="onSubmit"></form>

<!-- 修饰语可以使用链式书写 -->
<a @click.stop.prevent="doThat"></a>

<!-- 也可以只有修饰符 -->
<form @submit.prevent></form>

<!-- 仅当 event.target 是元素本身时才会触发事件处理器 -->
<!-- 例如：事件处理器不来自子元素 -->
<div @click.self="doThat">...</div>


<!-- 添加事件监听器时，使用 `capture` 捕获模式 -->
<!-- 例如：指向内部元素的事件，在被内部元素处理前，先被外部处理 -->
<div @click.capture="doThis">...</div>

<!-- 点击事件最多被触发一次 -->
<a @click.once="doThis"></a>

<!-- 滚动事件的默认行为 (scrolling) 将立即发生而非等待 `onScroll` 完成 -->
<!-- 以防其中包含 `event.preventDefault()` -->
<div @scroll.passive="onScroll">...</div>

<!-- 仅在 `key` 为 `Enter` 时调用 `submit` -->
<input @keyup.enter="submit" />
```

### `v-if `

只会在指令的表达式返回真值时才被渲染， 可以在 template 上使用

当 `v-if` 和 `v-for` 同时存在于一个元素上的时候，`v-if` 会首先被执行。

同时使用 `v-if` 和 `v-for` 是**不推荐的**，因为这样二者的优先级不明显。

```
<div v-if="type === 'A'">
  A
</div>
<div v-else-if="type === 'B'">
  B
</div>
<div v-else-if="type === 'C'">
  C
</div>
<div v-else>
  Not A/B/C
</div>
```

### `v-show`

`v-show` 会在 DOM 渲染中保留该元素；`v-show` 仅切换了该元素上名为 `display` 的 CSS 属性, 不支持在 template 上使用

### `v-for`

```
<li v-for="({ message }, index) in items">
  {{ message }} {{ index }}
</li>

<!-- 对象 -->
<li v-for="(value, key) in myObject">
  {{ key }}: {{ value }}
</li>

<!-- n 从 1 开始 -->
<span v-for="n in 10">{{ n }}</span>
```

事件, 使用 $event 或者 内联箭头函数

```
<!-- 使用特殊的 $event 变量 -->
<button @click="warn('Form cannot be submitted yet.', $event)">
  Submit
</button>

<!-- 使用内联箭头函数 -->
<button @click="(event) => warn('Form cannot be submitted yet.', event)">
  Submit
</button>
```

### v-model

```
<input
  :value="text"
  @input="event => text = event.target.value">
 
 <!-- 相当于上面的简化 -->
 <input v-model="text">
```



# 响应式基础

组合式 API 中，推荐使用 [`ref()`](https://cn.vuejs.org/api/reactivity-core.html#ref) 函数来声明响应式状态, `ref()` 接收参数，并将其包裹在一个带有 `.value` 属性的 ref 对象中返回

```
import { ref } from 'vue'

export default {
  setup() {
    const count = ref(0)

    function increment() {
      // 在 JavaScript 中需要 .value
      count.value++
    }

    // 不要忘记同时暴露 increment 函数
    return {
      count,
      increment
    }
  }
}
```

在模板中使用 ref 时，我们**不**需要附加 `.value`。为了方便起见，当在模板中使用时，ref 会自动解包

```
<button @click="increment">
  {{ count }}
</button>
```

`<script setup>`

在 `setup()` 函数中手动暴露大量的状态和方法非常繁琐。幸运的是，我们可以通过使用[单文件组件 (SFC)](https://cn.vuejs.org/guide/scaling-up/sfc.html) 来避免这种情况。我们可以使用 `<script setup>` 来大幅度地简化代码

```
<script setup>
import { ref } from 'vue'

const count = ref(0)

function increment() {
  count.value++
}
</script>

<template>
  <button @click="increment">
    {{ count }}
  </button>
</template>
```

`nextTick` 

修改响应式状态时，DOM 会被自动更新，但是 DOM 更新不是同步的， Vue 会在 next tick 更新周期中缓冲所有状态的修改，以确保不管进行了多少次状态修改，每个组件都只会被更新一次。

要等待 DOM 更新完成后在执行额外的代码， 可以使用 `nextTick()` 全局 API

```
import { nextTick } from 'vue'

async function increment() {
  count.value++
  await nextTick()
  // 现在 DOM 已经更新了
}
```

`reactive`

与将内部值包装在特殊对象中的 ref 不同， reactive() 将使对象本身具有响应性

reactivie() 返回的是一个原始对象的 Proxy, 和原始对象是不相等的，只有代理对象是响应式的，更改原始对象不会触发更新，为保证访问代理的一致性，对同一个原始对象调用 `reactive()` 会总是返回同样的代理对象，而对一个已存在的代理对象调用 `reactive()` 会返回其本身

```
import { reactive } from 'vue'

const state = reactive({ count: 0 })

<button @click="state.count++">
  {{ state.count }}
</button>
```

局限性

- 不能持有如 String 、 Number 或 Boolean 的原始类型，只能用于对象类型 对象，数组，Map, Set 这样的集合类型

- 不能替换整个对象，由于 Vue 的响应式追踪是通过属性访问实现的，因此我们必须始终保持对响应式对象的相同引用

  ```
  let state = reactive({ count: 0 })
  
  // 上面的 ({ count: 0 }) 引用将不再被追踪
  // (响应性连接已丢失！)
  state = reactive({ count: 1 })
  ```

- 对解构操作不友好

  ```
  const state = reactive({ count: 0 })
  
  // 当解构时，count 已经与 state.count 断开连接
  let { count } = state
  // 不会影响原始的 state
  count++
  
  // 该函数接收到的是一个普通的数字
  // 并且无法追踪 state.count 的变化
  // 我们必须传入整个对象以保持响应性
  callSomeFunction(state.count)
  ```

  

# 计算属性

若我们将同样的函数定义为一个方法而不是计算属性，两种方式在结果上确实是完全相同的，然而，不同之处在于**计算属性值会基于其响应式依赖被缓存**。一个计算属性仅会在其响应式依赖更新时才重新计算。这意味着只要 `author.books` 不改变，无论多少次访问 `publishedBooksMessage` 都会立即返回先前的计算结果，而不用重复执行 getter 函数。

相比之下，方法调用**总是**会在重渲染发生时再次执行函数。

```
<script setup>
import { reactive, computed } from 'vue'

const author = reactive({
  name: 'John Doe',
  books: [
    'Vue 2 - Advanced Guide',
    'Vue 3 - Basic Guide',
    'Vue 4 - The Mystery'
  ]
})

// 一个计算属性 ref
const publishedBooksMessage = computed(() => {
  return author.books.length > 0 ? 'Yes' : 'No'
})
</script>

<template>
  <p>Has published books:</p>
  <span>{{ publishedBooksMessage }}</span>
</template>
```

# 类与样式绑定

```
// 如果 active 是 true，添加 active class
<div
  class="static"
  :class="{ active: isActive, 'text-danger': hasError }"
></div>
```



```
const classObject = reactive({
  active: true,
  'text-danger': false
})

<div :class="classObject"></div>

<div class="active"></div>
```



在组件上使用

```
<!-- 子组件模板 -->
<p class="foo bar">Hi!</p>

<!-- 在使用组件时 -->
<MyComponent class="baz boo" />

<!-- 最终渲染 -->
<p class="foo bar baz boo">Hi!</p>
```

如果组件有多个根元素, `$attrs`

```
<!-- MyComponent 模板使用 $attrs 时 -->
<p :class="$attrs.class">Hi!</p>
<span>This is a child component</span>

<MyComponent class="baz" />

<p class="baz">Hi!</p>
<span>This is a child component</span>
```

# 生命周期

`onBeforeMount()`

在组件被挂载之前调用，组件已经完成了其响应式状态的设置，但还没有创建 DOM 节点，即将首次执行DOM 渲染过程。

`onMounted()`

注册一个回调函数，在组件挂载完成后执行

`onBeforeUpdate()`

在组件即将因为响应式状态变更而更新其DOM 树之前调用。

`onUpdated()`

注册一个回调函数，在组件因为响应式状态变更二更新其 DOM 树之后调用。

如果需要在某个特定的状态更改后访问更新后的 DOM， 使用 nextTick() 替代

不要在 updated 钩子中更改组件的状态，可能导致无限的更新循环

`onBeforeUnmount()`

在组件实例被卸载之前调用。

`onUnmounted()`

在组件实例被卸载之后调用

可以清理 计时器，DOM 事件监听器或者与服务器的连接。

`onErrorCaptured()`

注册一个钩子，在捕获了后代组件传递的错误时调用。

```
function onErrorCaptured(callback: ErrorCapturedHook): void

type ErrorCapturedHook = (
  err: unknown,
  instance: ComponentPublicInstance | null,
  info: string
) => boolean | void
```

`onActivated()`

注册一个回调函数，若组件实例是 [`Keepalive`](https://cn.vuejs.org/api/built-in-components.html#keepalive) 缓存树的一部分，当组件被插入到 DOM 中时调用。

`onDeactivated()`

注册一个回调函数，若组件实例是 [`Keepalive`](https://cn.vuejs.org/api/built-in-components.html#keepalive) 缓存树的一部分，当组件从 DOM 中被移除时调用。

