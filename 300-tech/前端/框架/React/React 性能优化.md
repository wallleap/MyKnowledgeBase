---
title: React 性能优化
date: 2024-08-22T02:22:48+08:00
updated: 2024-08-22T02:22:52+08:00
dg-publish: false
---

## React 性能优化思路

- 减少重新 render 的次数。
- 减少计算的量。主要是减少重复计算，对于函数式组件来说，每次 render 都会重新从头开始执行函数调用。
- 减少渲染的节点、降低渲染量
- 合理设计组件
- 在使用类组件（class）的时候，使用的 React 优化 API 主要是：shouldComponentUpdate 和 PureComponent，都是为了减少重新 render 的次数，主要是减少父组件更新而子组件也更新的情况
- 使用纯组件
- 使用 React.memo 进行组件记忆
- 使用 shouldComponentUpdate 避免重复渲染, 自定义浅比较逻辑
- 懒加载组件 (Suspense 和 lazy)
- 为组件创建错误边界
- 组件尽可能的进行拆分、解耦
- 使用不可突变数据结构
- Code Splitting
- 不要滥用 props, 只传需要的数据，避免多余的更新，尽量避免使用{...props}

### 谨慎使用 Context

Context 是跨组件传值的一种方案，但我们需要知道，我们无法阻止 Context 触发的 render。

不像 props 和 state，React 提供了 API 进行浅比较，避免无用的 render，Context 完全没有任何方案可以避免无用的渲染。

有几点关于 Context 的建议：

- Context 只放置必要的，关键的，被大多数组件所共享的状态。
- 对非常昂贵的组件，建议在父级获取 Context 数据，通过 props 传递进来。

## 函数组件优化方法

2.1 React.memo （减少 render 的次数）

> 可以减少重新 render 的次数，对标类组件里面的 PureComponent。

第一个参数是我们想要缓存复用的组件，第二个参数是一个比较函数，这个比较函数里我们可以控制 React.memo() 是否需要更新组件，如果比较函数返回 false 就更新组件，如果返回 true 那么 React.memo() 就返回上一次计算出来的组件。如果我们不传入 React.memo() 的第二个参数，默认会使用浅对比，因此我们只要使用 React.memo(C) ，优化效果与组件 C 继承 PureComponent 的写法是一样的 。

- 修改父组件 title 的时候同时传递给子组件一个 name 值。

```
// 父组件
import React, { useState } from "react";
import ReactDOM from "react-dom";
import Child from './child'

function Father() {
  const [title, setTitle] = useState("父组件的title");

  return (
    <div className="Father">
      <h1>{ title }</h1>
      <button onClick={() => setTitle("父组件的title改变了")}>修改父组件的title</button>
      <Child name="父组件传递给子组件的值"></Child>
    </div>
  );
}

// 子组件
import React from "react";

function Child(props) {
  console.log(props.name)
  return <p>{props.name}</p>
}

export default Child;

// 子组件
import React from "react";

function Child(props) {
  console.log(props.name)
  return <p>{props.name}</p>
}

export default React.memo(Child); // 用 React.memo()包裹

```

- 2.2 useCallback （减少 render 的次数）

2.3 useMemo （减少计算的量）
useMemo 主要是用来缓存计算量比较大的函数结果，可以避免不必要的重复计算，和 Vue 里面的 computed 有异曲同工的作用，可以减少计算的量。

PureComponent 会对 props 与 state 做浅比较，所以一定要保证 props 与 state 中的数据是 immutable 的。

如果你的数据不是 immutable 的，或许你可以自己手动通过 ShouldComponentUpdate 来进行深比较。当然深比较的性能一般都不好，不到万不得已，最好不要这样搞。

React.memo 会对 props 进行浅比较，如果一致，则不会再执行了。

React.memo 对 props 的变化做了优化，避免了无用的 render

React.useMemo 是 React 内置 Hooks 之一，主要为了解决函数组件在频繁 render 时，无差别频繁触发无用的昂贵计算 ，一般会作为性能优化的手段之一。

```
const App = (props)=>{
  const [boolean, setBoolean] = useState(false);
  const [start, setStart] = useState(0);

  // 这是一个非常耗时的计算
  const result = computeExpensiveFunc(start);
}

```

在上面例子中， computeExpensiveFunc 是一个非常耗时的计算，但是当我们触发 setBoolean 时，组件会重新渲染， computeExpensiveFunc 会执行一次。

这次执行是毫无意义的，因为 computeExpensiveFunc 的结果只与 start 有关系。

React.useMemo 就是为了解决这个问题诞生的，它可以指定只有当 start 变化时，才允许重新计算新的 result 。

```
const result = useMemo(()=>computeExpensiveFunc(start), [start]);

```

React.useCallback

```
const OtherComponent = React.memo(()=>{
    ...
});

const App = (props)=>{
  const [boolan, setBoolean] = useState(false);
  const [value, setValue] = useState(0);

  const onChange = (v)=>{
      axios.post(`/api?v=${v}&state=${state}`)
  }

  return (
    <div>
        {/* OtherComponent 是一个非常昂贵的组件 */}
        <OtherComponent onChange={onChange}/>
    </div>
  )
}

```

OtherComponent 是一个非常昂贵的组件，我们要避免无用的 render。虽然 OtherComponent 已经用 React.memo 包裹起来了，但在父组件每次触发 setBoolean 时， OtherComponent 仍会频繁 render。

因为父级组件 onChange 函数在每一次 render 时，都是新生成的，导致子组件浅比较失效。通过 React.useCallback，我们可以让 onChange 只有在 state 变化时，才重新生成。

```
const onChange = React.useCallback((v)=>{
  axios.post(`/api?v=${v}&state=${state}`)
}, [state])

```
