---
title: "Vue Record"
date: 2021-08-05T09:34:15+08:00
lastmod: 2021-08-05T09:34:15+08:00
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

Vue Study

<!--more-->

1. 路由跳转

   ```html
   <router-link :to="{ name: 'teamDetail', params: {id: record.id}}">
   	<a>Add User</a>
   </router-link>
   ```

2. Vue 父组件调用子组件方法

   ```html
   <template>
   	<child ref="child" />
   </template>
   <script>
   	name: 'parent',
   	components: {
   		Child
   	},
   	data () {
   		return {}
   	},
   	mounted () {
   		// 子组件 Child 中 有 refresh 方法
   		this.$refs.child.refresh(xxx)
   	}
   </script>
   ```

3. 子组件调用父组件的方法

   - 使用  `$parent.xxx` 调用
   
  ```html
     <template>
         <div>
             <button @click="sonFn">我是调用父组件方法的按钮</button>
         </div>
     </template>
      
     <script>
         export default {
             methods: {
                 sonFn () {
                     this.$parent.fatherFnOne() // 调用父组件的fatherFnOne方法
                 },
             },
         }
     </script>
     ```
   
   - 使用`$emit`向父组件触发一个事件，父组件监听这个事件
   
  ```html
     <!-- Parent -->
     <songCompoent @fatherFn="fatherFnTwo"></songCompoent>
     
     <!-- Child -->
     this.$emit('fatherFn')
     ```
   
   - 直接传入子组件调用

     
   
   