---
title: 052-React 组件
date: 2024-03-28T08:53:00+08:00
updated: 2024-08-21T10:32:38+08:00
dg-publish: false
---

能根其他物件**组**合起来的物**件**就是组件，组件并没有明确的定义，就目前而言

- 一个返回 React 元素的**函数**就是组件
- 在 Vue 里，一个**构造选项**就可以表示一个组件

## 元素与组件

```js
// React 元素，d 小写
const div = React.createElement('div', null, 'element')
// React 组件，D 大写
const Div = () => React.createElement('div', null, 'Component')
```

## React 两种组件

### 函数组件

```js
function Welcome(props) {
  return <h1>Hello, {{props.name}</h1>
}
// 箭头函数也可以 const Welcome = () => return ...
```

### 类组件

```js
class Welcome extends React.Component {
  /*
  constructor() {
    super()
    this.state = { n: 0 }
  }
  */
  render() {
    return <h1>Hello, {{this.props.name}</h1>
  }
}
```

使用的时候都是：

```js
<Welcome name="TOM" />
```

翻译成什么（可以使用 babel online 直接翻译查看）

- `<div />` 会被翻译成 `React.createElement('div')`
- `<Welcome />` 会被翻译成 `React.createElement(Welcome)`

`React.createElement` 的逻辑

- 如果传入一个**字符串** `'div'`，则会创建一个 `div`（虚拟 DOM 元素对象）
- 如果传入一个**函数**，则会调用该函数，**获取其返回值**
- 如果传入一个**类**，则在类前面加个 `new`（这回导致执行 constructor），获取一个组件**对象**，然后调用对象的 `render` 方法，获取其返回值

```js
import React from 'react'
import ReactDOM from 'react-dom'

function App() {
  return (
    <div className="App">
      <h2>App</h2>
      <Son />
    </div>
  )
}

class Son extends React.Component {
  constructor() {
    super()
    this.state = { // state
      n: 0
    }
  }
  add() { // 函数
    this.setState({ n: this.state.n + 1 })
  }
  render() {
    return (
      <div className="son">
        <h2>Son</h2>
        <p>n: {this.state.n}</p>
        <button onClick={() => this.add()}>+1</button>
        <Grandson />
      </div>
    )
  }
}

const Grandson = () => {
  const [n, setN] = React.useState(0)
  /*
  const arr = React.useState(0) // state 的初始值为 0
  const n = arr[0]
  const setN = arr[1] // set 的时候是新的 n
  */
  return (
    <div className="grandson">
      <h2>Grandson</h2>
      <p>n: {n}</p>
      <button onClick={() => setN(n + 1)}>+1</button>
    </div>
  )
}

ReactDOM.render(<App />, document.getElementById('root'))
```

## 使用 props（外部数据）

- 类组件直接读取属性 `this.props.xxx`
- 函数组件直接读取参数 `props.xxx`（形参可以是任意变量名，但一般叫 `props`）

```js
import React from 'react'
import ReactDOM from 'react-dom'

function App() {
  const a = "Hello, Son"
  return (
    <div className="App">
      <h2>App</h2>
      <Son msgForSon={a} />
    </div>
  )
}

class Son extends React.Component {
  render() {
    return (
      <div className="son">
        <h2>Son</h2>
        <p>App says, {this.props.msgForSon}</p>
        <Grandson msgForGrandson="Hello, Grandson" />
      </div>
    )
  }
}

const Grandson = props => {
  return (
    <div className="grandson">
      <h2>Grandson</h2>
      <p>Son says, {props.msgForGrandson}</p>
    </div>
  )
}

ReactDOM.render(<App />, document.getElementById('root'))
```

## 使用 state（内部数据）

类组件

- 用 `this.state` 读
- `this.setState` 写

函数组件用 `useState` 返回数组

- 第一项读
- 第二项写

```js
import React from 'react'
import ReactDOM from 'react-dom'

function App() {
  return (
    <div className="App">
      <h2>App</h2>
      <Son />
    </div>
  )
}

class Son extends React.Component {
  constructor() {
    super()
    // 初始化 state
    this.state = {
      n: 0
    }
  }
  add() {
    // 写
    // 直接 this.state.n += 1 不行，React 没有去监听它，并不会响应式修改视图，需要使用 setState，而且不推荐修改它，最好直接产生新的对象
    this.setState({ n: this.state.n + 1 })
    // 更推荐的是 this.setState(函数) conosle.log(this.state.n) 这个是旧的 n，在函数里就容易理解旧的（state.n），新的（const n = state.n + 1）
    /* this.setState((state, props) => {
      return { n: state.n + 1 }
    })
    */
  }
  render() {
    return (
      <div className="son">
        <h2>Son</h2>
        // 读取
        <p>n: {this.state.n}</p>
        <button onClick={() => this.add()}>+1</button>
        <Grandson />
      </div>
    )
  }
}

const Grandson = () => {
  const [n, setN] = React.useState(0)
  /*
  const arr = React.useState(0) // state 的初始值为 0
  const n = arr[0] // 第一项读
  const setN = arr[1] // 第二项写
  */
  return (
    <div className="grandson">
      <h2>Grandson</h2>
      <p>n: {n}</p>
      <button onClick={() => setN(n + 1)}>+1</button> /* setN 不会改变 n，会产生新的 n */
    </div>
  )
}

ReactDOM.render(<App />, document.getElementById('root'))
```

类组件注意事项

- `this.state.n += 1` 无效：其实 n 已经改变了，只不过 UI 不会自动更新而已；调用 `setState` 才会触发 UI 更新（异步更新）；因为 React 没有像 Vue 监听 data 一样监听 state
- `setState` 会异步更新 UI：`setState`
 之后，state 不会马上改变，立马读 state 会失败，更推荐的方式是 `setState(函数)`
 - `this.setState(this.state)` 不推荐：React 希望我们不要修改旧 state（**不可变数据**），常用代码：`setState({ n: state.n + 1 })`

总结：这是一种理念（**函数式**），喜欢就用，不喜欢就换 Vue

函数组件注意事项

- 通过 `setX(新值)` 来更新 UI
- 没有 `this`，一律用参数和变量

## 两种编程模型

![](https://cdn.wallleap.cn/img/pic/illustration/202403282322943.png)

**Vue 的编程模型**

一个对象对应一个虚拟 DOM，当对象的属性改变时，把属性相关的 DOM 节点全部更新

Vue 为了其他考量，也引入了虚拟 DOM 和 DOM diff（图中简化了）

**React 的编程模型**

一个对象对应一个虚拟 DOM，另一个对象对应另一个虚拟 DOM

对比两个虚拟 DOM，找不同（DOM diff），最后局部更新 DOM

## 复杂 state

类组件中，`setState` 只修改一个值，其他的值保留原样（不会设置 `undefined`，类似 `setState(...this.state, { n: this.state.n + 1 })`），会自动合并第一层属性，但是不会合并第二层属性

函数组件中 `setX` 完全不会帮你合并

```js
import React from 'react'
import ReactDOM from 'react-dom'

function App() {
  return (
    <div className="App">
      <h2>App</h2>
      <Son />
    </div>
  )
}

class Son extends React.Component {
  constructor() {
    super()
    // 初始化 state
    this.state = {
      n: 0,
      m: 0,
      user: {
        name: 'TOM',
        age: 18
      }
    }
  }
  addN() {
    // 写
    this.setState({ n: this.state.n + 1 })
  }
  addM() {
    this.setState({ m: this.state.m + 1 })
  }
  changeName() {
    this.setState({
      // 第二层需要使用 Object.assign 或 ... 运算符
      user: {
        ...this.state.user, // 没有的话 age 会置空
        name: 'Jack'
      }
    })
  }
  render() {
    return (
      <div className="son">
        <h2>Son</h2>
        // 读取
        <p>n: {this.state.n}</p>
        <button onClick={() => this.addN()}>+1</button>
        <p>m: {this.state.m}</p>
        <button onClick={() => this.addM()}>+1</button>
        <Grandson />
      </div>
    )
  }
}

const Grandson = () => {
  const [n, setN] = React.useState(0)
  const [m, setM] = React.useState(0)
  // 不推荐写法，setState 的时候需要自己手动 {...state, n: state.n + 1}
  // const [state, setState] = React.useState({n:0, m:0})
  return (
    <div className="grandson">
      <h2>Grandson</h2>
      <p>n: {n}</p>
      <button onClick={() => setN(n + 1)}>+1</button>
      <p>m: {m}</p>
      <button onClick={() => setM(m + 1)}>+1</button>
    </div>
  )
}

ReactDOM.render(<App />, document.getElementById('root'))
```

## 事件绑定

### 类组件的事件绑定

**推荐**：

```js
<button onClick={() => this.addN()}>+1</button>
```

分析其他写法：

```js
<button onClick={this.addN()}>+1</button>
// 这样会让 this.addN 里的 this 变成 window
<button onClick={this.addN.bind(this)}>+1</button>
// 这样可以，但是麻烦
this._addN = () => this.addN() // constructor 中
<button onClick={this._addN}>+1</button>
// 取了个别名，但是多声明了一次
constructor() {
  this.addN = () => this.setState({n: this.state.n + 1})
}
render() {
  return <button onClick={this.addN}>+1</button>
}
// 没有直接声明 addN 的
addN = () => this.setState({n: this.state.n + 1})
// 最终写法，不需要在 constructor 中
```

**最终写法**

```js
import React from 'react'
import ReactDOM from 'react-dom'

function App() {
  return (
    <div className="App">
      <h2>App</h2>
      <Son />
    </div>
  )
}

class Son extends React.Component {
  constructor() {
    super()
    // 初始化 state
    this.state = {
      n: 0,
      m: 0
    }
  }
  addN = () => this.setState({ n: this.state.n + 1 })
  addM() {
    this.setState({ m: this.state.m + 1 })
  }
  render() {
    return (
      <div className="son">
        <h2>Son</h2>
        // 读取
        <p>n: {this.state.n}</p>
        <button onClick={this.addN}>+1</button>
        <p>m: {this.state.m}</p>
        <button onClick={() => this.addM()}>+1</button>
      </div>
    )
  }
}

ReactDOM.render(<App />, document.getElementById('root'))
```

(1) `addN = () => this.setState({})` 和 (2) `addN() { this.setState({}) }` 写法的区别

(1) 等价于

```js
constructor() {
  this.addN = () => this.setState({})
}
```

(2) 等价于

```js
addN: function() {
  this.setState({})
}
```

(1) 在对象上，(2) 在原型上

(2) 的 this 可变成 window，而 (1) 是箭头函数则不会

总结：

- (1) 是对象**本身的属性**，这意味着每个 Son 组件都有自己的 addN，如果有两个 Son，就有两个 addN
- (2) 是对象的**共用属性**（也就是**原型上的属性**），这意味着所有 Son 组件共用一个 addN）

## React vs. Vue

相同点

- 都是对视图的封装，React 是用类和函数表示一个组件，而 Vue 是通过构造选项构造一个组件
- 都提供了 createElement 的 XML 简写，React 提供的是 JSX 语法，而 Vue 提供的是模版写法（语法多）

不同点

- React 是把 HTML 放在 JS 里写（HTML in JS）
- Vue 是把 JS 放在 HTML 里写（JS in HTML）
