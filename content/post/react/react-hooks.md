---
title: "React Hooks"
date: 2020-12-15T15:33:12+08:00
lastmod: 2020-12-15T15:33:12+08:00
draft: true
keywords: ["react","hooks"]
description: "React Hooks study"
tags: ["react"]
categories: ["react"]
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

React hooks

<!--more-->

# React Hooks

## useState

```html
import { useState } from "react"
const FuncComp = (props) => {
	const [count, setCount] = useState(0)
	return (<h3>Count: {count}<button onClick={() => setCount(count+1)}></button></h3>)
}
export default FuncComp
```

## useEffect

副作用，正常的一个组件应该只返回页面。componentDidUpdate() {}

```html
import React, { useState, useEffect } from "react"
const FuncComp = (props) => {
	const [count, setCount] = useState(0)
	// [count] 能够引起副作用的依赖，只有count发生变化时才会执行
	// 一个useEffect负责一件事
	useEffect(() => {
		console.log('useEffect', count)
		if (count % 2 === 0) {
			console.log('count', count);
		}
	}, [count])
	// 组件挂载 卸载
	useEffect(() => {
		// 组件挂载时执行
		console.log('挂载')
		const handleClick = () => console.log('handleClick')
		document.addEventListener('click', handleClick)
		// 如果[] 为空相当于组件被卸载时执行的操作。 componentDidUnmount()
		return () => {
			// 组件卸载时执行的操作
			console.log('卸载')
			document.removeEventListener('click', handleClick)
		}
	}, [])

	return (<h3>Count: {count}<button onClick={() => setCount(count+1)}></button></h3>)
}
export default FuncComp
```

## useReducer

```html
const [state, dispatch] = useReducer(reducer, initialArg, init);
```

useState`](https://zh-hans.reactjs.org/docs/hooks-reference.html#usestate) 的替代方案。它接收一个形如 `(state, action) => newState` 的 reducer，并返回当前的 state 以及与其配套的 `dispatch` 方法。

在某些场景下，`useReducer` 会比 `useState` 更适用，例如 state 逻辑较复杂且包含多个子值，或者下一个 state 依赖于之前的 state 等。并且，使用 `useReducer` 还能给那些会触发深更新的组件做性能优化，因为[你可以向子组件传递 `dispatch` 而不是回调函数](https://zh-hans.reactjs.org/docs/hooks-faq.html#how-to-avoid-passing-callbacks-down) 。

```html
import React, { useReducer } from "react"
const FuncComp = (props) => {
	const initialState = {count: 0, name: 'kyle'}
	const reducer = (state, action) => {
		// payload 默认1
		const {type, payload = 1} = action
		switch (type) {
 			case 'increment':
				// return 返回新对象, 要返回所有的值
             	return {...state, count: state.count + payload};
             case 'decrement':
                return {...state, count: state.count - 1};
             default:
                throw new Error();
 		}
	}
	const [state, dispatch] = useReducer(reducer, initialState)
	return (
        <>
          Count: {state.count}
          <button onClick={() => dispatch({type: 'decrement', payload: 2})}>-</button>
          <button onClick={() => dispatch({type: 'increment'})}>+</button>
        </>
  );
}
export default FuncComp
```

> React 会确保 `dispatch` 函数的标识是稳定的，并且不会在组件重新渲染时改变。这就是为什么可以安全地从 `useEffect` 或 `useCallback` 的依赖列表中省略 `dispatch`。

## [useContext](https://zh-hans.reactjs.org/docs/hooks-reference.html#usecontext)

深层值传递, 上下文

```html
const value = useContext(MyContext);
```

接收一个context 对象(`React.createContext`的返回值)，并返回该context的当前值。当前的context值由上层组件中距离当前组件最近的 `<MyContext.Provider>` 的 `value` prop决定。

当组件上层最近的 `<MyContext.Provider>` 更新时，该 Hook 会触发重渲染，并使用最新传递给 `MyContext` provider 的 context `value` 值。即使祖先使用 [`React.memo`](https://zh-hans.reactjs.org/docs/react-api.html#reactmemo) 或 [`shouldComponentUpdate`](https://zh-hans.reactjs.org/docs/react-component.html#shouldcomponentupdate)，也会在组件本身使用 `useContext` 时重新渲染。

调用了 `useContext` 的组件总会在 context 值变化时重新渲染。如果重渲染组件的开销较大，你可以 [通过使用 memoization 来优化](https://github.com/facebook/react/issues/15156#issuecomment-474590693)。

> `useContext(MyContext)` 只是让你能够*读取* context 的值以及订阅 context 的变化。你仍然需要在上层组件树中使用 `<MyContext.Provider>` 来为下层组件*提供* context。

```html
const ThemeContext = React.createContext('#000000');

function App() {
  const [bgColor, setBgColor] = useState('#000000')
  return (
	// value 要传值
	<input type="color" onChange={ev => setBgColor(ev.target.value)}></input>
	// 生产
    <ThemeContext.Provider value={bgColor}>
      <Toolbar />
    </ThemeContext.Provider>
  );
}

function Toolbar(props) {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

function ThemedButton() {
  // 消费
  const theme = useContext(ThemeContext);
  return (
    <button style={{ background: theme, color: theme }}>
      I am styled by theme context!
    </button>
  );
}
// 消费的第二种方式
function ThemedButton2() {
  return (
	<ThemeContext.Consumer>
        { value => {
        	<button style={{ background: value, color: value }}>
                I am styled by theme context!
            </button>
        }}
      
    </ThemeContext.Consumer>
    
  );
}
```

## useRef



## useCallback & useMemo

