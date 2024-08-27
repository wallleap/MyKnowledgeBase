---
title: React-useEffect 最佳实践
date: 2024-08-22T02:31:23+08:00
updated: 2024-08-22T02:31:32+08:00
dg-publish: false
---

> 在 function 组件中，每当 DOM 完成一次渲染，都会有对应的副作用执行，useEffect 用于提供自定义的执行内容，它的第一个参数（作为函数传入）就是自定义的执行内容。为了避免反复执行，传入第二个参数（由监听值组成的数组）作为比较 (浅比较) 变化的依赖，比较之后值都保持不变时，副作用逻辑就不再执行。

```ts
useEffect(() => {
  const subscription = props.source.subscribe();
  return () => {
    subscription.unsubscribe();
  };
}, [props.source]);
```

首先，我们要抛开生命周期的固有思维。**生命周期和 useEffect 是完全不同的。**

先说一下什么是副作用 effect：

本来我只是想渲染 DOM 而已，可是当 DOM 渲染完成之后，我还要执行一段别的逻辑。这一段别的逻辑就是副作用。

在 React 中，如果利用得好，副作用可以帮助我们达到更多目的，应对更为复杂的场景。利用不好也会变成灾难。

hooks 的设计中，每一次 DOM 渲染完成，都会有当次渲染的副作用可以执行。而 useEffect，是一种提供我们能够自定义副作用逻辑的方式。

深化一下使用场景：

如果除了在组件加载的那个时候会请求数据，在其他时刻，我们还想点击刷新或者下拉刷新数据，应该怎么办？

常规的思维是定义一个请求数据的方法，每次想要刷新的时候执行这个方法即可。而在 hooks 中的思维则不同：

**创造一个变量，来作为变化值，实现目的的同时防止循环执行**

```ts
import React, { useState, useEffect } from 'react';

export default function AnimateDemo() {
  const [list, setList] = useState(0);
  const [loading, setLoading] = useState(true);

  // DOM渲染完成之后副作用执行
  useEffect(() => {
    if (loading) {
      recordListApi().then(res => {
        setList(res.data);
      })
    }
  }, [loading]);

  return (
    <div className="container">
      <button onClick={() => setLoading(true)}>点击刷新</button>
      <FlatList data={list} />
    </div>
  )
}
```

还有一种情况：

外部想要控制内部的组件，就必须要往组件内部传入 props。而通常情况下，受控组件内部又自己有维护自己的状态。例如 input 组件。

也就意味着，我们需要通过某种方式，要将外部进入的 props 与内部状态的 state，转化为唯一数据源。这样才能没有冲突的控制状态变化。

换句话说，就是要利用 props，去修改内部的 state。

这是受控组件的核心思维。

```tsx
import React, { useState, useEffect } from 'react';

interface Props {
  value: number,
  onChange: (num: number) => any
}

export default function Counter({ value, onChange }: Props) {
  const [count, setCount] = useState<number>(0);

  useEffect(() => {
    value && setCount(value);
  }, [value]);

  return [
    <div key="a">{count}</div>,
    <button key="b" onClick={() => onChange(count + 1)}>
      点击+1
    </button>
  ]
}
```

如果有多个副作用时，相互之间没有关系，应该调用多个 useEffect()，而不应该合并写在一起。

```tsx
function App() {
  const [varA, setVarA] = useState(0);
  const [varB, setVarB] = useState(0);

  useEffect(() => {
    const timeout = setTimeout(() => setVarA(varA + 1), 1000);
    return () => clearTimeout(timeout);
  }, [varA]);

  useEffect(() => {
    const timeout = setTimeout(() => setVarB(varB + 2), 2000);

    return () => clearTimeout(timeout);
  }, [varB]);

  return <span>{varA}, {varB}</span>;
```

已上是我们开发过程中常见的用法。

最后来说一下

## useEffect 的执行顺序

React 的源码可以拆分为三块：

- 调度器：调度更新
- 协调器：决定更新的内容
- 渲染器：将更新的内容渲染到视图中

其中，只有渲染器会执行渲染视图操作。

对于浏览器环境来说，只有渲染器会执行类似 appendChild、insertBefore 这样的 DOM 操作。

## 协调器如何决定更新的内容

他会为需要更新的内容对应的 fiber（可以理解为虚拟 DOM）打上标记。

这些被打标记的 fiber 会形成一条链表 effectList。

渲染器会遍历 effectList，执行标记对应的操作。

- 比如 Placement 标记对应插入 DOM
- 比如 Update 标记对应更新 DOM 属性

useEffect 也遵循同样的工作原理：

1. 触发更新时，FunctionComponent 被执行，执行到 useEffect 时会判断他的第二个参数 deps 是否有变化。
2. 如果 deps 变化，则 useEffect 对应 FunctionComponent 的 fiber 会被打上 Passive（即：需要执行 useEffect）的标记。
3. 在渲染器中，遍历 effectList 过程中遍历到该 fiber 时，发现 Passive 标记，则依次执行该 useEffect 的 destroy（即 useEffect 回调函数的返回值函数）与 create（即 useEffect 回调函数）。

其中，前两步发生在协调器中。

所以，effectList 构建的顺序就是 useEffect 的执行顺序。

## effectList

协调器的工作流程是使用遍历实现的递归。所以可以分为递与归两个阶段。

我们知道，递是从根节点向下一直到叶子节点，归是从叶子节点一路向上到根节点。

effectList 的构建发生在归阶段。所以，effectList 的顺序也是从叶子节点一路向上。

useEffect 对应 fiber 作为 effectList 中的一个节点，他的调用逻辑也遵循归的流程。

## 不要用生命周期钩子类比 hook

我们在初学 hook 时，会用 ClassComponent 的生命周期钩子类比 hook 的执行时机。

即使官网也是这样教学的。

但是，从上文我们已经知道，React 的执行遵循：

调度 -- 协调 -- 渲染

渲染相关工作原理是按照：

构建 effectList -- 遍历 effectList 执行对应操作

整个过程都和生命周期钩子没有关系。

事实上生命周期钩子只是附着在这一流程上的钩子函数。

所以，更好的方式是从 React 运行流程来理解 useEffect 的执行时机。

## 渲染

按照流程，effectList 会在渲染器中被处理。

对于 useEffect 来说，遍历 effectList 时，会找到的所有包含 Passive 标记的 fiber。

依次执行对应 useEffect 的 destroy。

所有 destroy 执行完后，再依次执行所有 create。

整个过程是在页面渲染后异步执行的。

如果 useEffect 的 deps 为 []，由于 deps 不会改变，对应 fiber 只会在 mount 时被标记 Passive。

这点是类似 componentDidMount 的。

但是，处理 Passive effect 是在渲染完成后异步执行，而 componentDidMount 是在渲染完成后同步执行，所以他们是不同的。

渲染<App/>时控制台的打印顺序是？

```tsx
function Child() {
  useEffect(() => {
    console.log('child');
  }, [])

  return <p>hello</p>;
}

function Parent() {
  useEffect(() => {
    console.log('parent');
  }, [])

  return <Child/>;
}

function App() {
  useEffect(() => {
    console.log('app');
  }, [])

  return <Parent/>;
}
```

// child -> parent -> app
