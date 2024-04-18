---
title: 055-React Hooks 原理解析
date: 2024-03-29 17:27
updated: 2024-03-30 12:16
---

## 最简单的 useState 实现

分析：

首次渲染 render `<App />`

调用 App() 得到虚拟 Div 对象  创建真实 Div

用户点击 button 调用 setN(n + 1) 再次 render `<App />`

调用 App() 得到虚拟 Div 对象 DOM Diff 更新真实 Div

每次调用 App()，都会运行 useState(0)

- setN 的时候 n 不会变，App() 会重新执行
- App 重新执行，useState(0) 的时候，n 的值每次是不一样的（打印出来）
- setN
	- 一定会修改数据 x，把 n+1 存入 x
	- 一定会触发 `<App />` 重新渲染 re-render
- useState
	- 会从 x 中读取 n 的最新值
- x
	- 每个组件有自己的数据 x，将其命名为 state

尝试：

- 定义一个函数，接收初始值
- 需要有一个 state 接收新值或初始值
- 需要有个函数，设置新值且重新渲染
- 返回数组，第一项是 state，第二项是那个函数

```js
function reRender() {
  // 不 DOM Diff 直接全部渲染
  ReactDOM.render(<App />, document.getElementById('root'))
}

let _state // 每次都需要重新执行 useState1 所以这个值不能在里面定义
function useState1(initialVal) {
  _state = _state ?? initialVal // null 或 undefined 取后者
  const setState = (newVal) => {
    _state = newVal
    reRender()
  }
  return [_state, setState]
}
```

**当使用多次 useState1 的时候**，只有一个 `_state`

```js
const [n, setN] = useState1(0)
const [m, setM] = useState1(0)
```

改进思路：把 `_state`

- 改成对象，例如 `_state={n:0, m:0}`，因为不能提前知道变量名，所以排除
- 改成数组，例如 `_state=[0,0]`，貌似可以
	- 需要有个下标，表示第几个
	- 由于需要在 return 后让下标 + 1，所以引入中间变量当前下标
	- 重新渲染之前应该重置下标

```js
let _state = []
let index = 0

function reRender() {
  index = 0
  ReactDOM.render(<App />, document.getElementById('root'))
}

function useState1(initialVal) {
  let curIndex = index
  _state[curIndex] = _state[curIndex] ?? initialVal
  const setState = (newVal) => {
    _state[curIndex] = newVal
    reRender()
  }
  index += 1
  return [_state[curIndex], setState]
}
```

**不能在 if 中使用 useState**，因为用的数组对顺序要求很严格，第一次渲染和后面的渲染顺序应该一致

总结：

- 每个函数组件对应一个 React 节点
- 每个节点保存着 state 和 index
- useState 会读取 `state[index]`
- index 由 useState 出现的顺序决定
- setState 会修改 state，并触发更新
- 每次重新渲染，组件函数就会执行
- 对应的所有 state 都会出现“分身”
- 如果不希望出现分身，可以使用 useRef/useContext 等
