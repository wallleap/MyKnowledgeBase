---
title: 056-React Hooks 各个击破
date: 2024-03-29T05:30:00+08:00
updated: 2024-08-21T10:32:38+08:00
dg-publish: false
---

常用的 React Hooks

![](https://cdn.wallleap.cn/img/pic/illustration/202404021551377.png)

## 状态 useState

在 React 函数组件中默认只有属性，没有状态

**使用状态**

```js
const [count, setCount] = React.useState(0)
const [user, setUser] = React.useState({name: 'Tom'})
```

注意事项 1：**不可以局部更新**

如果 state 是一个对象，不能部分 setState，因为 setState 不会帮我们合并属性（后面 useReducer 也不会合并属性）

需要自己手动合并属性

```js
import {useState} from 'react'
// import ReactDOM from 'react-dom'
import ReactDOM from 'react-dom/client'

function App() {
  const [user, setUser] = useState({name: 'Tom', age: 18})
  const setUserName = () => {
    // 直接这样，user.age 丢失
    setUser({name: 'Jack'})
  }
  const changeUserName = () => {
    setUser({
      ...user,
      name: 'LiHua'
    })
  }
  return(
    <div>
      <h2>I'm {user.name}, {user.age} years old.</h2>
      <button onClick={setUserName}>Set User</button>
      <button onClick={changeUserName}>Change User</button>
    </div>
  )
}

// React 18 之后不再推荐使用 .render
// ReactDOM.render(<App />, document.getElementById('root'))
ReactDOM.createRoot(document.getElementById('root')).render(<App />)
```

注意事项 2：**地址要变**

setState(obj) 如果 obj 地址不变，那么 React 就认为数据没有变化（原来的 obj 变了，但是视图不更新）

```js
function App() {
  const [user, setUser] = useState({name: 'Tom', age: 18})
  const setUserName = () => {
    user.name = 'Jack'
    setUser(user)
    // 这样没用，需要传入一个新的对象
  }
  return(
    <div>
      <h2>I'm {user.name}, {user.age} years old.</h2>
      <button onClick={setUserName}>Set User</button>
    </div>
  )
}
```

useState 接收函数：该函数返回初始 state，且只执行一次（基本不用这个）

```js
const [state, setState] = useState(() => {
  return initialState
})
```

setState 接收函数（建议优先使用这种）

```js
setCount(i => i + 1)
```

## useReducer

> 用来践行 Flux/Redux 的思想

步骤：

1. 创建初始值 initialState
2. 创建所有操作 reducer(state, action)
3. 传给 useReducer，得到读和写 API
4. 调用写（`{type:'操作类型'}`）

例如：

```js
import { useReducer } from 'react'
import ReactDOM from 'react-dom/client'

// 1. 创建初始值 initialState
const initialState = {
  n: 22,
}

// 2. 创建所有操作 reducer(state, action)
const reducer = (state, action) => {
  // 可以改用 switch
  if (action.type === 'add') {
    return { n: state.n + action.count }
  } else if (action.type === 'multi') {
    return { n: state.n * action.count }
  } else if (action.type === 'minus') {
    return { n: state.n - action.count }
  } else {
    return state
  }
}

function App() {
  // 3. 传给 useReducer，得到读和写 API 一般这样命名
  const [state, dispatch] = useReducer(reducer, initialState) // 注意传参顺序
  const { n } = state

  // 4. 调用写（`{type:'操作类型'}`） 传的对象就是 action
  const add1 = () => dispatch({ type: 'add', count: 1 })
  const multi4 = () => dispatch({ type: 'multi', count: 4 })
  const minus1 = () => dispatch({ type: 'minus', count: 1 })
  return (
    <>
      {/* 5. 调用读 state.xxx */}
      <h1>n: {n}</h1>
      <button onClick={add1}>+1</button>
      <button onClick={multi4}>x4</button>
      <button onClick={minus1}>-1</button>
    </>
  )
}

ReactDOM.createRoot(document.getElementById('root')).render(<App />)
```

就相当于 useState，但是把所有的操作都聚拢在一起了

## useContext 上下文

上下文

- 全局变量是全局的上下文
- 上下文是局部的全局变量（在这个范围内是全局变量）

使用方法：

1. 使用 `C=createContext(initial)` 创建上下文
2. 使用 `<C.provider>` 圈定作用域
3. 在作用域内使用 `useContext(C)` 来使用上下文

```js
import ReactDOM from 'react-dom/client'
import { createContext, useContext, useState } from 'react'

// 1. 使用 `C=createContext(initial)` 创建上下文
const C = createContext(null)

function App() {
  const [count, setCount] = useState(0)

  // 2. 使用 `<C.provider>` 圈定作用域
  return (
    <C.Provider value={{count, setCount}}>
      <p>提供作用域 count: {count}</p>
      <button onClick={() => setCount(n => n + 1)}>+1</button>
      <br />
      <Child />
      <Grandchild />
    </C.Provider>
  )
}

function Child() {
  // 3. 在作用域内使用 `useContext(C)` 来使用上下文
  const {count, setCount} = useContext(C)
  return (
    <div>
      <p>Child count: {count}</p>
      <button onClick={() => setCount(n => n + 1)}>+1</button>
    </div>
  )
}

function Grandchild() {
  // 3. 在作用域内使用 `useContext(C)` 来使用上下文
  const {count, setCount} = useContext(C)
  return (
    <div>
      <p>Grandchild count: {count}</p>
      <button onClick={() => setCount(n => n + 1)}>+1</button>
    </div>
  )
}

ReactDOM.createRoot(document.getElementById('root')).render(<App />)
```

注意事项：

不是响应式的，而是自顶向下逐级通知使用到了就重新渲染的过程（在一个模块将 C 里面的值改变，另一个模块不会感知到这个变化）

## 使用 useReducer 和 useContext 代替 Redux

步骤：

1. 将数据集中在一个 store 对象
2. 将所有操作集中在 reducer
3. 创建一个 Context
4. 创建对数据的读写 API
5. 将 4 的内容放到 3 的 Context
6. 用 Context.Provider 将 Context 提供给所有组件
7. 各个组件用 useContext 获取读写 API

```js
import React, { createContext, useContext, useEffect, useReducer } from 'react'
import ReactDOM from 'react-dom/client'
/* import App from './App.jsx'
import './index.css' */

// 1. 将数据集中在一个 store 对象
const store = {
  user: null,
  books: null,
  movies: null
}

// 2. 将所有操作集中在 reducer
const reducer = (state, action) => {
  switch(action.type) {
    case 'setUser':
      return {
        ...state,
        user: action.payload
      }
    case 'setBooks':
      return {
        ...state,
        books: action.payload
      }
    case 'setMovies':
      return {
        ...state,
        movies: action.payload
      }
    default:
      throw new Error('Unhandled action type')
  }
}

// 3. 创建一个 Context
const Context = createContext(null)

function App() {
  // 4. 创建对数据的读写 API
  const [state, dispatch] = useReducer(reducer, store)

  // 5. 将 4 的内容放到 3 的 Context
  return (
    <Context.Provider value={{ state, dispatch }}>
      { /* 6. 用 Context.Provider 将 Context 提供给所有组件 */ }
      <User />
      <hr />
      <Books />
      <Movies />
    </Context.Provider>
  )
}

function User() {
  // 7. 各个组件用 useContext 获取读写 API
  const { state, dispatch } = useContext(Context)
  useEffect(() => {
    // 副作用写在这里，请求数据，只需要在组件加载时执行一次
    ajax('/user').then(payload => {
      // 写
      dispatch({ type: 'setUser', payload })
    })
  }, [])
  return (
    <div>
      <h2>个人信息</h2>
      <p>姓名：{ state.user?.name || '' }</p>
    </div>
  )
}

function Books() {
  // 7. 各个组件用 useContext 获取读写 API
  const { state, dispatch } = useContext(Context)
  useEffect(() => {
    ajax('/books').then(payload => {
      dispatch({ type: 'setBooks', payload })
    })
  }, [])

  return (
    <div>
      <h2>书籍</h2>
      <ul>
        {
          state.books ?
          state.books.map(book => <li key={book.id}>{ book.name }</li>) :
          null
        }
      </ul>
    </div>
  )
}

function Movies() {
  // 7. 各个组件用 useContext 获取读写 API
  const { state, dispatch } = useContext(Context)
  useEffect(() => {
    ajax('/movies').then(payload => {
      dispatch({ type: 'setMovies', payload })
    })
  }, [])

  return (
    <div>
      <h2>电影</h2>
      <ul>
        {
          state.movies ?
          state.movies.map(movie => <li key={movie.id}>{ movie.name }</li>) :
          null
        }
      </ul>
    </div>
  )
}

/**
 * 写个 ajax 函数，模拟请求
 * @param {string} url
 * @returns {Promise<any>}
 *  */
const ajax = (url) => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      switch(url) {
        case '/user':
          resolve({
            id: 1,
            name: '张三',
            age: 18
          })
          break
        case '/books':
          resolve([
            {
              id: 1,
              name: '西游记'
            },
            {
              id: 2,
              name: '红楼梦'
            }
          ])
          break
        case '/movies':
          resolve([
            {
              id: 1,
              name: '电影1'
            },
            {
              id: 2,
              name: '电影2'
            }
          ])
          break
        default:
          reject('404')
      }
    }, 3000)
  })
}

ReactDOM.createRoot(document.getElementById('root')).render(<App />)
```

编程技巧：表驱动

```js
const userReducer = {
  setUser: (state, action) => {
    return { ...state, user: action.payload }
  },
  updateUser: (state, action) => { },
}
const booksReducer = {
  setBooks: (state, action) => {
    return { ...state, books: action.payload }
  },
  delBook: (state, action) => { },
}
const moviesReducer = {
  setMovies: (state, action) => {
    return { ...state, movies: action.payload }
  },
  delMovie: (state, action) => { },
}

// 把所有的操作写成一个对象，操作名对应操作函数
const reducerMap = {
  ...userReducer,
  ...booksReducer,
  ...moviesReducer
}

const reducer = (state, action) => {
  const fn = reducerMap[action.type]
  if(fn) {
    return fn(state, action)
  } else {
    throw new Error('Unhandled action type')
  }
}
```

## 副作用 useEffect

副作用

- 对环境的改变即为副作用，例如修改 document.title
- 但我们不一定非要把副作用放到 useEffect 里
- 实际上叫做 afterRender 更好，每次 render 后执行

用途

- 作为 componentDidMount 使用，第二参数为 `[]`
- 作为 componentDidUpdate 使用，可指定依赖
- 作为 componentWillUnmount 使用，通过 `return`
- 以上三种用途可同时存在

特点：如果同时存在多个 useEffect，会按照出现次序执行

```js
import { useEffect, useState } from 'react'

function App() {
  const [effectVisible, setEffectVisible] = useState(false)
  return (
    <>
      <button onClick={ () => setEffectVisible(!effectVisible) }>切换显示</button>
      {effectVisible && <EffectDemo />}
    </>
  )
}

function EffectDemo() {
  const [count, setCount] = useState(0)
  const [user, setUser] = useState({
    name: 'Tom',
    age: 18
  })
  useEffect(() => {
    console.log('第一次渲染执行')
  }, []) // [] 里面的变化时候执行，为空时只执行一次
  useEffect(() => {
    console.log('count 变化执行')
  }, [count]) // count 变化执行
  useEffect(() => {
    console.log('user 变化执行')
  }, [user])
  useEffect(() => {
    console.log('每次渲染/变化执行')
  }) // 任何一个 state 变化时都执行
  useEffect(() => {
    // 副作用（任意除了计算外的操作）
    document.title = `点击了 ${count} 次`
    const timerId = setInterval(() => {
      setCount(count + 1)
    }, 1000)
    return () => {
      // 清除副作用
      document.title = 'Title'
      clearInterval(timerId)
    }
  })
  return(
    <div>
      <h2>Count: {count}, Name: {user.name}, Age: {user.age}</h2>
      <button onClick={() => setCount(i => i + 1)}>+1</button>
      <input type="text" value={user.name} onChange={e => setUser(name => ({...user, name: e.target.value}))} />
    </div>
  )
}

ReactDOM.createRoot(document.getElementById('root')).render(<App />)
```

## 布局副作用 useLayoutEffect

布局副作用

- useEffect 在浏览器**渲染完后**执行
- useLayoutEffect 在浏览器**渲染前**执行

```js
function App() {
  const [count, setCount] = React.useState(0)
  React.useLayoutEffect(() => {
    document.querySelector('#x'.innerText = 'count: 1000')
  }, [count])
  return(
    <div id="x" onClick={() => setCount(0)}>count: [{count}]</div>
  )
}
```

特点

- useLayoutEffect 总是比 useEffect 先执行（通过打印时间查看）
- useLayoutEffect 里的任务最好影响了 Layout

```js
function App() {
  const [count, setCount] = React.useState(0)
  const time = React.useRef(null)
  const onClick = () => {
    setN(i => i + 1)
    time.current = performance.now()
  }
  // 改成 useEffect 试下
  React.useLayoutEffect(() => {
    if(time.current) {
      console.log(perfromance.now() - time.current)
    }
  })
  return(
    <div class="App">
      <h2>count: [{count}]</h2>
      <button onClick={onClick}>Click</button>
    </div>
  )
}
```

经验：为了用户体验，优先使用 useEffect（优先渲染）

## 记忆 useMemo 和 useCallback

看代码：

```js
import ReactDOM from 'react-dom/client'
import { useState } from 'react'

function App() {
  const [n, setN] = useState(0)
  const [m, setM] = useState(100)
  return (
    <>
      <button onClick={() => setN(n => n + 1)}>Click {n}</button>
      <br />
      <Child data={m} />
    </>
  )
}

function Child(props) {
  console.log('Child 执行了')
  // 假设这里有大量代码
  return (
    <p>Child: {props.data}</p>
  )
}

ReactDOM.createRoot(document.getElementById('root')).render(<App />)
```

点击按钮的时候，n 改变了，App 执行没问题，但是发现 Child 也执行了

优化技巧：使用 memo 把 Child 再封装一下，这样 Child 就只会在它 props 变化的时候渲染

```js
function App() {
  const [n, setN] = useState(0)
  const [m, setM] = useState(100)
  return (
    <>
      <button onClick={() => setN(n => n + 1)}>Click {n}</button>
      <br />
      {/* <Child data={m} /> */}
+     <Child2 data={m} />
    </>
  )
}

function Child(props) {
  console.log('Child 执行了')
  // 假设这里有大量代码
  return (
    <p>Child: {props.data}</p>
  )
}

const Child2 = React.memo(Child)
```

现在点击按钮的时候 Child2 就不会重新执行了，修改按钮 setM 发现 Child2 重新渲染

Child 只用了一次，所以直接放到 memo 里

```js
const Child = React.memo(props => {
  console.log('Child 执行了')
  // 假设这里有大量代码
  return (
    <p>Child: {props.data}</p>
  )
})
```

- React 有默认的多余的 render
- 把代码中的 Child 用 React.memo(Child) 代替，如果 props 不变，就没有必要再次执行一个函数组件

但是，如果添加了监听函数之后

```js
import ReactDOM from 'react-dom/client'
import { useState } from 'react'

function App() {
  const [n, setN] = useState(0)
  const [m, setM] = useState(100)
  // 添加了一个空函数
  const onClickChild = () => {}
  return (
    <>
      <button onClick={() => setN(n => n + 1)}>Click {n}</button>
      <br />
      // 通过 props 传递
      <Child data={m} onClick={onClickChild} />
    </>
  )
}

const Child = React.memo(props => {
  console.log('Child 执行了')
  // 假设这里有大量代码
  return (
    // 在这里调用
    <p onClick={props.onClick}>Child: {props.data}</p>
  )
})

ReactDOM.createRoot(document.getElementById('root')).render(<App />)
```

现在再点击 App 中的按钮，Child 也会重新执行

App 重新执行，`const onClickChild = () => {}` 这句也重新执行，是不同的空函数（新旧函数虽然功能一样，但是地址不一样），导致 props 变了，所以 Child 会重新执行，可以使用 useMemo 解决

useMemo 用于缓存新旧组件迭代的时候，使用上次的值

- 第一个参数为函数，返回需要缓存的内容，这里是一个空函数 `() => value`
- 第二个参数是个数组，指定变化的依赖项

```js
const onClickChild = useMemo(() => {
  return () => {}
}, [m])
```

- 只有当依赖变化的时候才会计算出新的 value
- 如果依赖不变，那么就重用之前的 value（有点像 Vue 2 的 computed）

如果 value 是个函数，就需要写成 `useMemo(() => x => console.log(x))`，这是一个返回函数的函数，可以使用 useCallback 解决，它第一个参数可以只写返回的那个函数

```js
const onClickChild = useCallback(() => {
  console.log(m)
}, [m])
```

语法糖，`useCallback(x => log(x), [x])` 等价于 `useMemo(() => x => log(x), [x])`

## 引用 useRef

目的：

- 让一个值在组件不断 reder 时**保持不变**
- 初始化：`const count = useRef(0)`
- 读取：`count.current`
- 使用 `.current` 可以确保两次 useRef 的是同一个值（引用）

```js
import { useEffect, useRef, useState } from "react"

window.num = 0 // 渲染了多少次

function App() {
  window.num++ // 使用全局变量
  console.log(window.num)
  const count = useRef(0) // 使用 ref
  const [n, setN] = useState(0)
  useEffect(() => {
    count.current += 1
    console.log('渲染次数：', count.current)
  })
  return (
    <button onClick={() => setN(i => i + 2)}>n: {n}</button>
  )
}
```

useRef 不会在变化时自动 render

- 不符合 React 的理念，他的理念是 `UI = f(data)`，它给了更多的自由
- 如果想要这个功能，可以自己加，监听 ref（代理/或其他手段），当 ref.current 变化时，调用 setX 即可（useState 变化会更新视图）

## useState 会生成“分身”，useRef 不会自动 render

useState 会自动生成新的 n

```js
import ReactDOM from 'react-dom/client'
import { useState } from 'react'

function App() {
  const [n, setN] = useState(0)
  const log = () => setTimeout(() => console.log(n), 3000)
  return (
    <>
      <h2>{n}</h2>
      <button onClick={() => setN(i => i + 1)}>+1</button>
      <button onClick={log}>log</button>
    </>
  )
}

ReactDOM.createRoot(document.getElementById('root')).render(<App />)
```

通过时间差可以看出生成了新的 n（**React** 的函数式它使用变量，不倾向于修改它）

- 先点击加按钮，再点击 log 按钮，正常打印出 1
- 但是先点击 log，再点击加按钮，打印出来的还是 0

![](https://cdn.wallleap.cn/img/pic/illustration/202404021403594.png)

那想要一个强制的贯穿始终的状态怎么办

- 全局变量，`window.xxx`
- 使用 useRef
- 使用 useContext

```js
import ReactDOM from 'react-dom/client'
import { useRef, useState } from 'react'

function App() {
  const nRef = useRef(0)
  const log = () => setTimeout(() => console.log(nRef.current), 3000)
  // React 中没有可以直接触发更新的，但是 useState 的 state 改变可以更新
  const update =useState({})[1] // 不要第一项，因为没用到

  return (
    <>
      <h2>{nRef.current} 没有实时更新</h2>
      <button onClick={() => {nRef.current += 1;update({})}>+1</button>
      <button onClick={log}>log</button>
    </>
  )
}

ReactDOM.createRoot(document.getElementById('root')).render(<App />)
```

nRef 一直是同一个，使用 setN 进行更新

useRef 贯穿一个组件的前中后

```js
import ReactDOM from 'react-dom/client'
import { createContext, useContext, useState } from 'react'

const themeContext = createContext(null) // 局部的全局变量

function App() {
  const [theme, setTheme] = useState('red')

  return (
    // 传入一个对象，同名缩写
    <themeContext.Provider value={{ theme, setTheme }}>
      <div style={{color: `${theme}`}}>
        <h2>{theme}</h2>
        <ChildA />
        <ChildB />
      </div>
    </themeContext.Provider>
  )
}

function ChildA() {
  const { setTheme } = React.useContext(themeContext) // 析构
  return <button onClick={() => setTheme('red')}>改成红色</button>
}

function ChildB() {
  const { setTheme } = React.useContext(themeContext)
  return <button onClick={() => setTheme('yellow')}>改成黄色</button>
}

ReactDOM.createRoot(document.getElementById('root')).render(<App />)
```

## forwardRef

使用 ref 引用到 DOM 对象（类组件可以直接使用）

```js
import ReactDOM from 'react-dom/client'
import { forwardRef, useRef } from 'react'

function App() {
  const buttonRef = useRef(null)
  return (
    <Button ref={buttonRef} className='red' />
  )
}

const Button = forwardRef((props, ref) => { // 使用 forwardRef 让它可以接收另一个参数
  console.log(props, ref) // props 中不包含 ref
  return <button ref={ref} {...props}>按钮</button> {/* ref 传递 */}
})

ReactDOM.createRoot(document.getElementById('root')).render(<App />)
```

useRef 可以用来引用 DOM 对象，也可以引用普通对象

由于 props 不包含 ref（大部分时候不需要），所以需要用 forwardRef 包一层

## useImperativeHandle

应该叫 setRef

```js
import ReactDOM from 'react-dom/client'
import { forwardRef, useEffect, useImperativeHandle, useRef } from 'react'
import { createRef } from 'react'

function App() {
  const buttonRef = useRef(null)
  useEffect(() => {
    console.log(buttonRef.current) // 打印出来的是 <button class="red">按钮</button>
    // 封装之后是个对象 { realButton: {}, x: () => {} }
  })
  return (
    <>
      <button onClick={() => buttonRef.current.x()}>x</button>
      <Button ref={buttonRef} className='red' />
    </>
  )
}

const Button = forwardRef((props, ref) => {
  const realButton = createRef(null)
  useImperativeHandle(ref, () => ({
    x: () => {
      realButton.current.remove()
    },
    realButton
  }))
  return <button ref={ref} {...props}>按钮</button>
})

ReactDOM.createRoot(document.getElementById('root')).render(<App />)
```

正常打印的是 button 对象，现在返回一个封装了的对象

## 自定义 Hook

看下这个

```js
import ReactDOM from 'react-dom/client'
import { useState, useEffect } from 'react'

function App() {
  const { list, setList } = useList()
  return (
    <>
      <h2>List 渲染</h2>
      {
        list ? (
          <ol>
            {
              list.map(item => (
                <li key={item.id}>{item.name}</li>
              ))
            }
          </ol>
        ) : ('加载中……')
      }
    </>
  )
}

// 实现一下 useList
function useList() {
  const [list, setList] = useState(null)
  useEffect(() => {
    ajax().then(list => setList(list))
  }, [])
  return { list, setList }
}

/* 假数据 */
function ajax() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve([
        { id: 1, name: 'Tom' },
        { id: 2, name: 'Jack' },
        { id: 3, name: 'Alice' },
        { id: 4, name: 'Bob' }
      ])
    }, 2000)
  })
}

ReactDOM.createRoot(document.getElementById('root')).render(<App />)
```

一般把 hook 放到 `src/hooks` 目录下，文件名为 `useList.js`（hook 以 `use` 开头），暴露一下读写接口

升级一下，增加其他接口

```js
function useList() {
  const [list, setList] = useState(null)
  useEffect(() => {
    ajax().then(list => setList(list))
  }, [])
  return {
    list,
    setList,
    addItem: name => {
      setList([...list, { id: Math.random(), name }])
    },
    deleteItem: id => {
      setList(list.filter(item => item.id !== id))
    }
  }
}
```

尽量使用自定义 Hook，不要直接在组件上方用 useState 等

## stale closure 过时闭包

[Be Aware of Stale Closures when Using React Hooks (dmitripavlutin.com)](https://dmitripavlutin.com/react-hooks-stale-closures/)

```js
function createIncrement(incBy) {
  let value = 0;

  function increment() {
    value += incBy;
    console.log(value)
    
    const message = `Current value is ${value}`;
    return function log() {
      console.log(message);
    }
  }
  
  return increment
}

const c = createIncrement(1);
const log = c(); // 1
c(); // 2
c(); // 3
// Does not work!
log();       // "Current value is 1"
```

解决：

1、重新获取 `const latestLog = c()`（最新的 log）

2、把 `const message` 放到 log 里（最新的 value）

```js
return function log() {
  const message = `Current value is ${value}`;
  console.log(message);
}
```

useEffect 中

```js
function WatchCount() {
  const [count, setCount] = useState(0);

  useEffect(function() {
    const id = setInterval(function log() {
      console.log(`Count is: ${count}`) // 获取的一直是旧 count
    }, 2000);
  }, []) // 空依赖

  return (
    <div>
      {count}
      <button onClick={() => setCount(count + 1) }>
        Increase
      </button>
    </div>
  );
}
```

解决：要打印它就把它放到依赖中 `[count]`（注意：好习惯需要在 willUnmount 的时候 `clearInterval(id)`）

useState 也是，推荐使用的是 setN 里放函数，`setN(i => i + 1)`
