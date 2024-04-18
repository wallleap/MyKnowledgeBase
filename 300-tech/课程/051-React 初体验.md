---
title: 051-React 初体验
date: 2023-10-09 10:46
updated: 2024-03-28 20:31
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - React 初体验
rating: 1
tags:
  - web
  - React
category: web
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
description: 文章描述
source: 
url: //myblog.wallleap.cn/post/1
---

React 学习会比 Vue 更简单，因为记的东西会比较少，主要是对 JS 的熟练程度

## 引入 React

有两种方式：CDN 引入或 import

**通过 CDN 引入，需要注意顺序**

```html
<div id="root"></div>
<!-- 先引入 React -->
<script src="https://.../umd/react.development.js"></script> <!-- 或 xxx.min.js -->
<!-- 再引入 ReactDOM -->
<script src="https://.../umd/react-dom.development.js"></script>

<!-- 打印看下 -->
<script>
cosnole.log(window.React)
cosnole.log(window.React.createElement)
cosnole.log(window.ReactDOM)
cosnole.log(window.ReactDOM.render)
</script>
```

cjs 和 umd 的区别

- cjs 全称 CommonJS，是 Node.js 支持的模块规范
- umd 是同一模块定义，兼容各种模块规范（包含浏览器）
- 理论上有优先使用 umd，同时支持 Node.js 和浏览器
- 最新的模块规范是使用 `import` 和 `export` 关键字

**通过 webpack 引入 React**

安装依赖

```sh
yarn add react react-dom
```

`import ... from ...` 方式引入

```js
import React from 'react'
import ReactDOM from 'react-dom'
```

- 注意大小写，尽量保持一致（约定俗成）
- 除了 webpack 外，rollup、parcel 也支持上面写法

老手使用 webpack/rollup 等构建工具，新手直接使用 **create-react-app** 等脚手架

```sh
yarn global add create-react-app
create-react-app react-demo
cd react-demo
yarn start
```

代码结构：

```
|---public/
    |---index.html
|---src/
    |---index.js
    |---App.js
```

## 用 React 实现 +1

### 简单的写法

```js
let n = 0
const root = document.querySelector('#root')
const App = React.createElement('div', {className='red'}, n)

ReactDOM.render(App, root)
```

> createElement 的返回值 element 可以代表一个 div，但 element 并不是真正的 div，一般称 element 为**虚拟 DOM 对象**

第一个参数是标签名字符串，第二个参数是属性对象，更多的操作（值、更多元素）都写在第三个参数中

为了实现一个 +1 我们可能会写出以下代码：

```js
let n = 0
const root = document.querySelector('#root')
const App = React.createElement('div', {className='red'}, [
  n,
  React.createElement('button',{
    onClick: () => {
      n += 1
    }
  }, '+1')
])

ReactDOM.render(App, root)
```

现在点击按钮会发现并没有变化，它没有自己刷新

为了让它刷新我们可能会让他重新渲染一次

```js
let n = 0
const root = document.querySelector('#root')
const App = React.createElement('div', {className='red'}, [
  n,
  React.createElement('button',{
    onClick: () => {
      n += 1
      ReactDOM.render(App, root)
    }
  }, '+1')
])

ReactDOM.render(App, root)
```

这会发现还是没有效果，原因是 `App = React.createElement()` 只执行了一次

### 用 for 和 setTimeout 举例

在 for 外面声明变量

```js
let i // 等价于在 for 中声明 var i（变量提升）
for (i = 0; i < 6; i++) {
  setTimeout(()=>console.log(i), 1000)
}
```

只关注 console.log 会发现将在 1s **后**同时输出 6 个 6

- setTimeout 会在 n 毫秒**后尽快**执行（其他同步代码执行完成后空闲就执行）
- 在外部声明的变量始终是同一个，在跳出 for 循环的时候，i 已经为 6 了
- 可以在内部重新声明 `let j = i`，再输出 j 的值，这样由于 let 会生成块级作用域，每次的 j 都是新的，这样会在 1s 后同时输出 0-5

```js
let i
for (i = 0; i < 6; i++) {
  let j = i
  setTimeout(()=>console.log(j), 1000)
}
// for(let i = 0; i < 6; i++)
```

由于输出 0-5 才是符合开发者认知的，所以后面可以直接在 for 初始化的时候，直接使用 let，这样每次都会生成一个新的 i，打印的时候就和重新声明 j 效果一样

那还有其他方式吗

```js
let i
for (i = 0; i < 6; i++) {
  !function(j){
    setTimeout(()=>console.log(j), 1000)
  }(i)
}
```

使用立即执行函数，每次进这个循环体都会把 i 传进立即执行函数（立即执行函数的参数也可以改成 i），这样每次都是新的 i，所以能输出 0-5

### 函数组件

App 是一个函数，函数执行的时候会重新读 n 的值

```js
let n = 0
const root = document.querySelector('#root')
const App = () => React.createElement('div', {className='red'}, [
  n,
  React.createElement('button',{
    onClick: () => {
      n += 1
      ReactDOM.render(App(), root)
    }
  }, '+1')
])

ReactDOM.render(App(), root)
```

> `()=> React 元素` 返回 element 的函数，也可以代表一个 div，这个函数可以**多次执行**，每次得到最新的虚拟 div，React 会对比两个虚拟 div，==找出不同==，局部更新视图（找不同的算法叫做 **DOM Diff** 算法）

### 函数的本质——延迟

React 促使我们思考函数的本质

**普通代码**

```js
let b = a + 1
```

**函数**

```js
let f = () => a + 1
let b = f()
```

可以看出

- 普通代码**立即求值**，读取 a 的**当前**值
- 函数会等调用时再求值，即**延迟求值**，且求值时才会读取 a 的**最新**值

### 对比 React 元素和函数组件

- `App1 = React.createElement('div', null, n)`，App1 是一个 React 元素
- `App2 = () => React.createElement('div', null, n)`，App2 是一个 React 函数组件，函数 App2 是**延迟执行**的代码，会在**被调用的时候**执行

## JSX 的用法

X 代表扩展，JSX 就是 JS 扩展版

JSX 是为了弥补写 React 太丑而引入的语言

### Vue 和 React 比较一

- Vue 中有 vue-loader，在 `.vue` 文件中写 `<template>`、`<script>`、`<style>`，通过 vue-loader 变成一个构造选项
- React 有 JSX，可以吧 `<button onClick="add">+1</button>` 变成 `React.createElement('button', { onClick: ... }, '+1')`
	- 这里面使用了编译原理
	- 猜测是使用 jsx-loader 实现，然而实际上 jsx-loader 被 babel-loader 取代了，jsx-loader 被 webpack 内置了

### 使用 JSX

**方法一：使用 CDN**

```html
<!-- 引入 babel.js  -->
<script src=".../babel.min.js"></script>
<!-- 把 script 标签上 type 属性改为 text/bael，babel 会自动进行转译  -->
<script type="text/babel"></script>
```

不会在生产环境使用这个方法，因为效率低

- 需要下载 `babel.min.js`
- 要在浏览器端把 JSX 翻译成 JS（这两件事都可以在 build 的时候做）

```js
let n = 0
const App = () => (
  <div className="red">
    {n}
    <button onClick={()=>{n+=1;render()}}>+1</button>
  </div>
)
const render = () => ReactDOM.render(<App />, document.querySelector('#root'))
render()
```

**方法二：webpack + babel-loader**

新手跳过，直接用方法三

**方法三：使用 create-react-app**

`App.js` 中默认就使用了 JSX 语法，webpack 让 JS 默认走 babel-loader

`index.html`

```html
<div id="root"></div>
```

`App.js`

```js
import React from 'react' // 后面相当于 React.createElement

const App = () => {
  return ( // 这里这个 () 不能省略，否则就是 return undefined 了
    <div>App 组件</div>
  )
}

export default App
```

`index.js`

```js
import React from 'react'
import ReactDOM from 'react-dom'
import App from './App.js'

ReactDOM.render(<App />, document.getElementByID('root'))
```

使用 JSX 的注意事项：

- 直接写标签
- class 还是直接在标签中写 className 属性
- 标签里面的所有 JS 代码都要使用 `{}` 括起来
- 习惯 `return` 后面加 `()`

### Vue 和 React 比较二

都可以写 HTML

- Vue 写在 `.vue` 文件的 `<template>` 里
- React 把 HTML 混在 JS/JSX 文件里

### 条件判断

在 Vue 中 `v-if`、`v-else-if`、`v-else`

在 React 中

```js
const Component = () => {
  return n % 2 === 0 ? <div>n 是偶数</div> : <span>n 是奇数</span>
}
```

如果外面是 div，则 JS 代码需要放到 `{}` 中，return 后加 `()`

```js
const Component = () => {
  return (
    <div>
      { n % 2 === 0 ? <div>n 是偶数</div> : <span>n 是奇数</span> }
    </div>
  )
}
```

还可以把这些写成变量，最后 return

```js
const Component = () => {
  const content = (
    <div>
      { n % 2 === 0 ? <div>n 是偶数</div> : <span>n 是奇数</span> }
    </div>
  )
  return content
}
```

还可以单独提里面的

```js
const Component = () => {
  const inner = n % 2 === 0 ? <div>n 是偶数</div> : <span>n 是奇数</span>
  const content = (
    <div>
      { inner }
    </div>
  )
  return content
}
```

就想用 if-else 也可以

```js
const Component = () => {
  let inner
  if(n % 2 === 0)
    inner = <div>n 是偶数</div>
  else 
    <span>n 是奇数</span>
  const content = (
    <div>
      { inner }
    </div>
  )
  return content
}
```

- 在 Vue 中只能用它提供的语法写条件判断
- 在 React 中想怎么写就怎么写，就是在写 JS

### 循环语句

在 Vue 中可以遍历数组和对象，使用 `v-for` 并且需要添加一个 `:key`

在 React 中

需要接收 props

```js
const Component = (props) => {
  return props.numbers.map((n, index) => {
    return <div>下标 {index} 值为 {n}</div>
  })
}
```

外面有 div

```js
const Component = (props) => {
  return (<div>
    { props.numbers.map((n, index) => {
      return <div>下标 {index} 值为 {n}</div>
    }) }
  </div>)
}
```

也可以用 for 循环，能 push div

```js
const Component = (props) => {
  const arr = []
  for(let i = 0; i < props.numbers.length; i++) {
    arr.push(<div>下标 {i} 值为 {props.numbers[i]}</div>)
  }
  return <div>{ arr }</div>
}
```

使用 Vue 封装好的 `v-*` 还是使用 React 里写原生 JS

偷懒的人和新手喜欢 Vue
