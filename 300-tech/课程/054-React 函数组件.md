---
title: 054-React 函数组件
date: 2024-03-29 00:25
updated: 2024-04-15 14:33
---

函数组件比类组件更简单一点，且完全可以代替 class 组件，它还消除了 this

## 创建方式

```js
const Hello = (props) => {
  return (<div>{props.msg}</div>)
}
// 省略
const Hello = props => <div>{props.msg}</div>
// 普通函数
function Hello(props) {
  return <div>{props.msg}</div>
}
```

注释：

```js
function MyComponent() {
  return (
    // Comment around JSX
    <div>
      {/* Comment inside JSX */}
      <Hello // comment inside JSX tag
        message="Hello, World!" // comment inside JSX tag
      /> 
    </div>
  )
}
```

## 函数组件代替 class 组件

面临两个问题，纯函数只能用于数据计算，不涉及计算的操作

- 函数组件没有 state
- 函数组件没有生命周期

解决：React v16.8.0 推出 Hook API，其中

- useState 可以解决 state（之前已经写过代码）
- useEffect 可以解决通用的副作用（不涉及计算的操作，例如事件监听或订阅、日志生成、获取数据、数据存储、改变应用状态、改变 DOM 等）

Hook（钩子）约定以 `use` 开头

### 函数组件模拟生命周期

useState 状态钩子

- 参数为初始值
- 返回一个数组
	- 第一项是一个变量，指向状态的当前值（例如 `count`）
	- 第二项是一个函数，用来更新状态，约定是 `set` 前缀加上状态的变量名，例如 `setCount`

useEffect 有两个参数

- 第一个参数是函数，里面写“副作用”（渲染/变化了会调用）
- 第二个参数是个数组，里面写副作用函数的依赖项，只有依赖项改变了才会重新渲染
	- 第二个参数没写，用到的所有的变量都依赖（相当于全写在数组中）
	- 空数组，代表没有依赖项，只有在第一次加载进入的时候执行一次，后面组件重新渲染，不会执行
- useEffect 允许返回一个函数，这个函数用来清理副作用，在组件卸载的时候会执行该函数（例如 setTimeout、订阅等）

```js
const Hello = props => {
  const [n, setN] = React.useState(0)
  const [childVisible, setChildVisible] = React.useState(true)
  const onClick = () => {
    setN(n + 1)
  }
  const hide = () => {
    setChildVisible(false)
  }
  const show = () => {
    setChildVisible(true)
  }
  // 直接写这里是每次渲染都会调用
  console.log('渲染了一次', n)
  // 只在第一次调用
  useEffect(()=>{
    console.log('1')
  }, [])
  // 变的时候
  useEffect(()=>{
    console.log('n 变了')
  }, [n]) // 数据写上依赖的变量
  useEffect(()=>{
    console.log('渲染或者变化')
  }) // 所有的都变了就把 [] 去掉
  return (<div>
    <h2>{n}</h2>
    <button onClick={onClick}>+1</button>
    { childVisible ?
      <button onClick="hide">hide</button> :
      <button onClick="show">show</button> }
    { childVisible ? <Child /> : null }
  </div>)
}

const Child = props => {
  // 销毁
  React.useEffect(() => {
    return () => {
      console.log('Child 销毁了')
    }
  })
  return <div>Child</div>
}
```

模拟 componentDidMount

```js
useEffect(() => {
  console.log('第一次渲染')
}, [])
```

模拟 componentDidUpdate

```js
useEffect(() => {
  console.log('任意属性变更')
})
useEffect(() => {
  console.log('n 变了')
}, [n])
```

模拟 componentWillUnmount

```js
useEffect(() => {
  console.log('第一次渲染')
  return () => {
    console.log('组件要死了')
  }
})
```

constructor 不需要模拟，在函数组件执行的时候，就相当于 constructor

shouldComponentUpdate 依靠 React.memo 和 useMemo 解决

render 函数组件的返回值就是 render 的返回值

### 自定义 Hook 之 useUpdate

由于使用 useEffect 第一次的时候也会触发

```js
const [nUpdateCount, setNUpdateCount] = useState(0)

useEffect(()=> {
  setNUpdateCount(x => x+1)
}, [n])
useEffect(()=>{
  if(nUpdateCount > 1) {
    console.log('n 变了')
  }
}, [nUpdateCount])
```

- 自定义 Hook 以 `use` 开头
- 把这些代码放到函数里，想办法传入需要的内容
- 按照控制台说的消除警告
- 仿照 useEffect

```js
// 可以放到函数组件外
const useUpdate = (fn, arr) => {
  if (!Array.isArray(arr) || !(fn instanceof Function))
    return
  const [count, setCount] = useState(0)
  useEffect(() => {
    setCount(x => x + 1)
  }, arr)
  useEffect(() => {
    if (count > 1) {
      fn()
    }
  }, [count, fn])
}
// 使用
useUpdate(() => console.log('n 变了'), n)
```
