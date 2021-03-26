---
title: "Js Standard"
date: 2021-03-26T20:38:29+08:00
lastmod: 2021-03-26T20:38:29+08:00
draft: false
keywords: ["js"]
description: "js standard"
tags: ["js"]
categories: ["js"]
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
js 书写规范
<!--more-->
# [js 书写规范](https://github.com/lin-123/javascript#types--primitives)

1. 复制使用`const`，避免使用`var`, 或者使用 `let`

   > 确保不会改变初始值，重复引用会导致bug和代码难以理解, `let`和`const`都是块级作用域

   ```javascript
   let a = 1;
   const b = 1;
   ```

2. 使用字面值创建对象

   ```javascript
   // bad
   const item = new Object();
   const items = new Array();
   // good
   const item = {};
   const items = [];
   // 数组使用 push 方法添加
   // bad
   items[items.length] = 'aaa';
   // good
   items.push('aaa');
   ```

3. 创建动态属性名的对象时，用计算后属性名

   ```javascript
   function getKey(k) {
     return `a key named ${k}`;
   }
   // bad
   const obj = {
     id: 1,
     name: 'kyle',
   }
   obj[getKey['age']] = 15;
   // good
   const obj = {
     id: 1,
     name: 'kyle',
     [getKey('age')]: 15,
   };
   ```

4. 用对象方法简写

   ```javascript
   // bad
   const atom = {
     value: 1,
     addValue: function(value) {
       return atom.value + value;
     },
   };
   // good
   const atom = {
     value: 1,
     addValue(value) {
       return atom.value + value;
     },
   }
   ```

5. 用属性值缩写,并将所有缩写放在对象声明的开始

   > 为了更方便的知道有哪些属性用了缩写

   ```javascript
   const a = 'aa';
   // bad
   const obj = {
     b: 'b',
     a: a,
   }
   // good
   const obj = {
     a,
     b: 'a',
   }
   ```

6. 不要直接调用`Object.prototype`上的方法，如`hasOwnProperty`, `propertyIsEnumerable`, `isPrototypeOf`。

   > 在一些有问题的对象上，这些方法可能会被屏蔽掉 - 如： `{hasOwnProperty: false}` 或这是一个空对象 `Object.create(null)`

   ```javascript
   // bad
   console.log(object.hasOwnProperty(key));
   // good
   console.log(Object.prototype.hasOwnProperty.call(object, key));
   
   // best
   // 在模块作用内做一次缓存
   const has = Object.prototype.hasOwnProperty;
   /* or */
   import has from 'has';// https://www.npmjs.com/package/has
   // ...
   console.log(has.call(object, key));
   ```

7. 对象浅拷贝时，推荐用 `...` 运算符

   ```javascript
   // 浅拷贝
   const copy = {...obj, c: 3};
   // rest 赋值运算符
   // copy => { a: 1, b: 2, c: 3 }
   const {a, ...obj} = copy; // obj => {b: 2, c: 3}
   ```

8. 用 `...` 运算符而不是`Array.from`来将一个可迭代的对象转换成数组

   ```javascript
   const foo = document.querySelectorAll('.foo');
   // good
   const nodes = Array.from(foo);
   // best
   const nodes = [...foo];
   ```

9. 用`Array.from` 去将一个类数组对象转换成一个数组。

   ```javascript
   const arr = {0: 'foo', 1: 'bar', length: 2};
   // bad
   const array = Array.prototype.slice.call(arr);
   // good
   const array = Array.from(arr);
   ```

10. 用`Array.from` 而不是 `...` 运算符去做map遍历，因为这样可以避免创建一个临时数组。

    ```javascript
    // bad
    const baz = [...foo].map(bar);
    // good
    const baz = Array.from(foo, bar);
    ```

11. 用对象的解构赋值来获取和使用对象某个或多个属性值

    ```javascript
    // bad
    function getName(user) {
      const firtName = user.firstName;
      const lastName = user.lastName;
      return `${firstName} ${lastName}`;
    }
    // good
    function getName(user) {
      const {firtName, lastName} = user
      return `${firstName} ${lastName}`;
    }
    // best
    function getName({firstName, lastName}) {
      return `${firstName} ${lastName}`;
    }
    ```

12. 用数组解构

    ```javascript
    const arr = [1,2,3,4];
    // bad
    const first = arr[0];
    const second = arr[1];
    
    // good
    const [first, second] = arr;
    ```

13. 多个返回值用对象的解构，而不是数组解构

    ```javascript
    // bad
    function process(input) {
      return [left, right, top, bottom];
    }
    // 调用者需要知道返回值的顺序
    const [left, __, top] = process(input);
    
    // good
    function process(input) {
      return {left, right, top, bottom};
    }
    // 调用者返回想用的值就可以
    const {left, top} = process(input);
    
    ```

14. 永远不要在字符串中用 `eval()`, 潘多拉盒子

15. 不要使用不必要的转义字符

    > 反斜线可读性差，所以他们只在必须使用时才出现

    ```javascript
    // bad
    const foo = '\'this\' \i\s \"quoted\"';
    // good
    const foo = '\'this\' is "quoted"';
    //best
    const foo = `my name is '${name}'`;
    ```

16. 用命名函数表达式而不是函数声明

    > 函数表达式: const func = function () {}

    > 函数声明： function func() {}

    ```javascript
    // bad
    function foo() {
      // ...
    }
    
    // bad
    const foo = function () {
      // ...
    }
    
    // good
    // lexical name distinguished from the variable-referenced invocation(s)
    // 函数表达式名和声明的函数名是不一样的
    const short = function longUniqueMoreDescriptiveLexicalFoo() {
      // ...
    };
    ```

17. 把立即执行函数包裹在园括号里

    ```javascript
    (function () {
      ...
    })
    ```

18. 不要在非函数块(if, while 等)内声明函数。把这个函数分配给一个变量。

19. [`block`]的定义是：一系列的语句；但是函数声明不是一个语句。函数表达式是一个语句。

    ```javascript
    // bad
    if (user) {
      function test() {
        console.log('Nope.')
      }
    }
    ```

20. 不要用 `arguments` 命名参数，他的优先级高于每个函数作用域自带的 `arguments` 对象，这会导致函数自带的 `arguments` 值被覆盖

    ```javascript
    // bad
    function foo(name, options, arguments) {
      // ...
    }
    
    // good
    function foo(name, options, args) {
      // ...
    }
    ```

21. 不要使用 `arguments`, 用rest语法 `...` 代替

    ```javascript
    // bad
    function concatenateAll() {
      const args = Array.prototype.slice.call(arguments);
      return args.join('');
    }
    
    // good
    function concatenateAll(...args) {
      return args.join('');
    }
    ```

22. 用默认参数语法而不是在函数里对参数重新赋值, 把默认参数赋值放在最后

    ```javascript
    // really bad
    function handleThings(opts) {
      // 不， 我们不该改arguments
      // 第二： 如果 opts 的值为 false, 它会被赋值为 {}
      // 虽然你想这么写， 但是这个会带来一些细微的bug
      opts = opts || {};
      // ...
    }
    
    // still bad
    function handleThings(opts) {
      if (opts === void 0) {
        opts = {};
      }
      // ...
    }
    
    // bad
    function handleThings(opts = {}, name) {
      // ...
    }
    
    // good
    function handleThings(name, opts = {}) {
      // ...
    }
    ```

23. 不要改参数

    ```javascript
    // bad
    function f1(obj) {
      obj.key = 1;
    };
    
    // good
    function f2(obj) {
      const key = Object.prototype.hasOwnProperty.call(obj, 'key') ? obj.key : 1;
    }
    ```

24. 使用箭头表达式

    > 他创建了一个 `this` 的当前执行上下文的函数的版本，箭头函数是更简洁的语法

    ```javascript
    // bad
    [1, 2, 3].map(function (x) {
      const y = x + 1;
      return x * y;
    });
    
    // good
    [1, 2, 3].map((x) => {
      const y = x + 1;
      return x * y;
    });
    ```

25. 如果函数体由一个没有副作用的表达式语句组成，删除大括号和return

    ```javascript
    let bool = false;
    
    // bad
    // 这种情况会return bool = true, 不好
    foo(() => bool = true);
    
    // good
    foo(() => {
      bool = true;
    });
    ```

26. 表达式涉及多行,把他包裹在圆括号里更可读

    ```javascript
    // bad
    ['get', 'post', 'put'].map(httpMethod => Object.prototype.hasOwnProperty.call(
        httpMagicObjectWithAVeryLongName,
        httpMethod
      )
    );
    
    // good
    ['get', 'post', 'put'].map(httpMethod => (
      Object.prototype.hasOwnProperty.call(
        httpMagicObjectWithAVeryLongName,
        httpMethod
      )
    ));
    ```

27. 常用 `class`, 避免直接操作 `prototype`

    ```
    // bad
    function Queue(contents = []) {
      this.queue = [...contents];
    }
    Queue.prototype.pop = function () {
      const value = this.queue[0];
      this.queue.splice(0, 1);
      return value;
    };
    
    // good
    class Queue {
      constructor(contents = []) {
        this.queue = [...contents];
      }
      pop() {
        const value = this.queue[0];
        this.queue.splice(0, 1);
        return value;
      }
    }
    ```

    

