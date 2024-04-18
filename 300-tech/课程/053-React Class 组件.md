---
title: 053-React Class 组件
date: 2024-03-29 00:25
updated: 2024-03-29 16:00
---

## 两种创建 class 组件的方式

### ES 5 方式（过时）

```js
import React from 'react'

const A = React.createClass({
  render() {
    return (
      <div>A Component</div>
    )
  }
})

export default A
```

由于 ES 5 不支持 class 关键字，所以才会有这种方式

### ES 6 方式

```js
import React from 'react'

class B extends React.Component {
  constructor(props) {
    super(props)
  }
  render() {
    return (
      <div>B Component</div>
    )
  }
}

export default B
```

以后就只使用这种方式创建类组件，可以用 webpack + babel 将 ES 6 翻译成 ES 5

## props 外部数据（属性）

```js
import React from 'react'
import B from './B.js'

class Parent extends React.Component {
  constructor(props) {
    super(props)
    this.state = { name: 'Tom' }
  }
  onClick = () => {}
  render() {
    return (
      <B
        name={this.state.name}
        onClick={this.onClick} // 这里的 onClick 是一个回调
      >Hi</B>
    )
  }
}

export default Parent
```

写在组件上的 props 外部数据会被包装成一个对象：

```js
{
  name: 'Tom',
  onClick: ...,
  children: 'Hi'
}
```

### 初始化

初始化之后，`this.props` 就是外部数据**对象的地址**了

```js
class B extends React.Component {
  constructor(props) { // 要么不初始化，即不写 constructor
    super(props) // 要么写全套，不写 super 直接报错
  }
  render() {}
}

export default B
```

### 读取

通过 `this.props.xxx` 读取

```js
class B extends React.Component {
  constructor(props) {
    super(props)
  }
  render() {
+    return (<div onClick={this.props.onClick}>
+      {this.props.name} {this.props.children}
+    </div>)
  }
}

export default B
```

### 写（不允许）

> **不准改写 props**

非得写的方式：

- 改 props 的值（地址）：`this.props = {}` 新的对象
	- 不要写这样的代码，没有意义
	- 既然是外部数据就应当由外部更新
- 改 props 的属性：`this.props.xxx = 'hi'`
	- 没有意义
	- 既然是外部数据就不应该从内部改值

原则：应该由数据的主人对数据进行更改

### 相关钩子

`componentWillReceiveProps` 当接受新的 props 时，会触发此钩子

```js
import React from 'react'

class Parent extends React.Component {
  constructor(props) {
    super(props)
    this.state = { a: 1 }
  }
  onClick = () => {
    this.setState({ a: this.state.a + 1 })
  }
  render() {
    return (
      <B
        num={this.state.a}
        onClick={this.onClick}
      >Hi</B>
    )
  }
}

class B extends React.Component {
  componentWillReceiveProps(newProps, nextContext) {
    console.log('props changed', newProps)
  }
  render() {
    return (<div>{this.props.num}</div>)
  }
}
```

该钩子已经被弃用，更名为 `UNSAFE_componentWillReceiveProps`，现在不要使用这个钩子

### 作用

- 接受外部数据
	- 只能读不能写
	- 外部数据由父组件传递
- 接受外部函数
	- 在恰当的时机调用该函数
	- 该函数一般是父组件的函数

## state & setState 内部数据

### 初始化

```js
class B extends React.Component {
  constructor(props) {
    super(props)
+   this.state = {
+    user: { name: 'Tom', age: 18 }
+   }
  }
  render() {}
}
```

### 读写

- 读用 `this.state.xxx.yyy.zzz`
- 写用 `this.setState(newState, fn)`
	- setState 不会立刻改变 `this.state`，会在当前代码运行完后，再去更新 `this.state`，从而触发 UI 更新
	- `this.setState((state, props)=>newState, fn)` 这样更容易理解
	- fn 会在写入成功后执行
- 写的时候会 shallow merge（会自动将新 state 和旧 state 进行一级合并）

```js
add = () => {
  this.setState({ x: this.state.x + 1 }) // 还是原来的值
}
// 尽量这下面这种方式
add2 = () => {
  this.setState((state) => ({ x: state.x + 1 }))
}
// 不推荐下面的用法
add3 = () => {
  this.state.x += 1
  this.setState(this.state)
}
```

## 生命周期

类比如下代码

```js
// 这是 divEL 的 create/construct 过程
const divEl = document.createElement('div')
// 这是初始化 state
divEl.textContent = 'Hi'
// 这是 divEl 的 mount 过程
document.body.appendChild(divEl)
// 这是 divEl 的 update 过程
divEl.textContent = 'Hello'
// 这是 divEl 的 unmount 过程
divEl.remove()
```

同理，React 组件也有这些过程，我们称之为生命周期

### 生命周期函数列表

xxx 之后会执行这个函数

- **constructor()** 构造
	- 在这里初始化 state
- static getDerivedStateFromProps()
- **shouldComponentUpdate()** 应该更新
	- return false 阻止更新（另一分支应当有 return true）
- **render()** 渲染
	- 创建虚拟 DOM
- getSnapshotBeforeUpdate()
- **componentDidMount()** 挂载完
	- 组件已经出现在页面
- **componentDidUpdate()** 更新完
	- 组件已更新
- **componentWillUnmount()** 将卸载
	- 组件将死
- static getDerivedStateFromError()
- componentDidCatch()

### constructor

用途：

- 初始化 props
- 初始化 state，但此时不能调用 setState
- 用来写 bind this

```js
constructor() {
  super()
  this.onClick = this.onClick.bind(this)
}

// 用新语法代替：
constructor() {
  super()
}
onClick = () => {}
```

### shouldComponentUpdate

用途：它允许我们**手动判断**是否要进行组件更新，我们可以根据应用场景**灵活地设置返回值**，以**避免不必要的更新**

- 返回 true 表示不阻止 UI 更新
- 返回 false 表示阻止 UI 更新

```js
class App extends React.Component {
  constructor(props) {
    super(props)
    this.state = {
      n: 1
    }
  }
  add = () => {
    this.setState(state => ({n: state.n + 1}))
    this.setState(state => ({n: state.n - 1}))
    // {n: 1} 和 {n: 1} 不是同一个对象
  }
  shouldComponentUpdate(newProps, newState) {
    if(newState.n === this.state.n) // 阻止更新
      return false
    else
      return true
  }
  render() {
    console.log('render') // 重复执行了
    return (<div>
      <h2>{this.state.n}</h2>
      <button onClick="add">+1</button>
    </div>)
  }
}
```

启发：

- 可以把 newState 和 this.state 的每个属性都对比一下，如果全都相等就不更新，如果有一个不等就更新
- React 内置了这个功能：React.PureComponent 代替 React.Component

PureComponent 会在 render 之前对比新 state 和旧 state 的每一个 key，以及新 props 和旧 props 的每一个 key。

如果所有 key 的值全都一样，就不会 render；如果有任何一个 key 的值不同，就会 render。

### render

用途：

- 展示视图 `return (<div>Hi</div>)` （返回的是虚拟 DOM 对象）
- 只能有一个根元素，多个元素可以使用 Fragment `<React.Fragment></React.Fragment>` 包裹，可以省略写 `<></>`

技巧：

- render 里面可以写 if-else、三元运算符 (A ? B : C) 等
- render 里不能直接写 for 循环，需要用数组，可以写 array.map（循环），循环的元素上应该加上 key 属性

### componentDidMount

用途：

- 在元素插入页面后执行代码，这些代码依赖 DOM
- 比如想获取 div 的高度，最好在这里写（获取到元素 ）
- 在这可以发起加载数据的 AJAX 请求（官方推荐）
- 首次渲染会执行此钩子

```js
class App extends React.Component {
  divRef = undefined
  constructor(props) {
    super(props)
    this.state = {
      n: 1,
      width: undefined
    }
    // 1. 引用
    this.divRef = React.createRef()
  }
  componentDidMount() {
    // 3. 读
    const div = thisdivRef.current
    const { width } = div.getBoundingClientRect()
    this.setState({ width })
  }
  render() {
    return (<div>
      <h2>{this.state.n}</h2>
      // 2. 关联
      <p ref={this.divRef}>{this.state.width}px</p>
    </div>)
  }
}
```

### componentDidUpdate

用途：

- 在视图更新后执行代码
- 此处也可以发起 AJAX 请求，用于**更新数据**（比如 userId 变了重新请求用户数据）
- 首次渲染**不会**执行此钩子
- 在此处 setState 可能会引起无限循环，除非放在 if 里
- 若 shouldComponentUpdate 返回 false 则不触发此钩子

参数 componentDidUpdate(prevProps, prevState, snapshot)

### componentWillUnmount

用途：

- 组件将要被移出页面然后被销毁时执行
- unmount 过的组件不会再次 mount

举例：谁污染谁治理原则

- 如果在 componentDidMount 中监听了 window.scroll，那么需要在这里取消监听
- 如果在挂载后创建了 Timer，那么就要在这里取消 Timer
- 如果在挂载后创建了 AJAX 请求，并且数据未传回，那么就要在这里取消请求

### 分阶段看

首次渲染：constructor --> render - 更新 UI -> componentDidMount

再次渲染：props 变了/setState/forceUpdate --> shouldComponentUpdate --> 结束 或者 render - 更新 UI -> componentDidUpdate

销毁：componentWillUnmount
