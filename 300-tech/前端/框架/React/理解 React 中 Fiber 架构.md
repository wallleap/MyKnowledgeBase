---
title: 背景
date: 2024-08-22T03:51:47+08:00
updated: 2024-08-22T03:51:58+08:00
dg-publish: false
---

## 背景

自从 React16 版本更新了 Hook 用法，同时引入了新的 Fiber 架构去重构整个渲染和事件处理过程，React 团队引入 Hook 是为了更好剥离业务代码，让开发能更加友好的抽象代码，达到低耦合的函数组件目的，那么重构 Diff 算法，引入 Fiber 架构是为了什么呢？ 其实只是为了能够一个目标 `快速响应`，原先 Diff 算法时间复杂度为<span>$$ O(n^3)$$</span> ，最后经过 Fiber 重构达到了 $$ O(n) $$

，这里面具体有什么门道，值得我们去深入研究一下。

<!-- more -->

## 问题

在了解 Fiber 架构之前，我们需要对原有 React16 之前版本是有什么问题，才需要引入 Fiber 架构去解决该问题？

React15 及以前的版本采用的是 Stack Reconciler（栈协调器）架构，使用同步递归方式去创建虚拟 DOM，一旦进入创建过程，就无法中断，如果创建过程超过 16ms，用户就会出现页面卡顿感觉。具体可以参考下图：

![20230117-1](https://github.com/peng-yin/note/assets/25680922/c9b8f554-d317-46e6-a2e9-58574ca6c0dc)

因此，从网上搜索了一下 React15 及以前的版本反馈，的主要问题有如下几个：

- React 的动画效果表现不佳
- React 在有大量 DOM 节点渲染卡顿

### 为什么

为什么会出现卡顿的情况，主要原因如下：

1. JavaScript 是单线程，与渲染线程互斥，当其中一个线程执行时，另一个线程只能挂起等待。
2. Stack Reconciler 栈协调器某个任务是长期占用 JavaScript 主线程

## 前置知识

为了更好了解 Fiber 架构设计，需要提前了解一些前置知识，每个知识点其实都需要深入了解，这里只是简单描述，主要有以下几点：

- 单线程的 JavaScript 与多线程的浏览器
- React 生命周期
- React 虚拟 DOM

### 单线程的 JavaScript 与多线程的浏览器

在我们学习前端知识的时候，有个结论是： ` 单线程的 JavaScript 与多线程的浏览器 `。

一个完整的 web 网页在浏览器显示和交互的进程（chrome 为主），需要涉及到线程主要以下几个部分：

- `GUI 渲染线程`，负责渲染浏览器界面 HTML 元素,当界面需要重绘 (Repaint) 或由于某种操作引发回流 (reflow) 时,该线程就会执行。
- `JavaScript引擎线程`，JS 内核，负责处理 Javascript 脚本程序。 一直等待着任务队列中任务的到来，然后解析 Javascript 脚本，运行代码。
- `定时触发器线程`，定时器 setInterval 与 setTimeout 所在线程，为什么要单独弄个线程处理定时器？是因为 JavaScript 引擎是单线程的, 如果处于阻塞线程状态就会影响记计时的准确
- `事件触发线程`，用来控制事件轮询，JS 引擎自己忙不过来，需要浏览器另开线程协助
- `异步http请求线程`，在 `XMLHttpRequest` 或 `fetch` 在连接后是通过浏览器新开一个线程请求， 将检测到状态变更时，如果设置有回调函数，异步线程就产生状态变更事件放到 JavaScript 引擎的处理队列中等待处理。这里需要注意 `XMLHttpRequest` 和 `fetch` 的区别，`fetch` 是 w3c 标准化后一个专门提供给开发调用发起 http 的 API 接口，XMLHttpRequest 是一个非标准化的 Http 请求对象，主要是可以发起 http 请求获取 XML 数据。

上述就是浏览器的多线程，然后单线程的 JavaScript 通常指的是 `JavaScript引擎线程`，为什么需要单线程？因为多线程可能会出现各种 UI 交互冲突问题。因此了解单线程 JS 需要注意几点：

- GUI 线程和 JS 引擎是互斥的，当 JS 引擎执行时 GUI 线程会被挂起，GUI 更新则会被保存在一个队列中等到 JS 引擎线程空闲时立即被执行。
- JS 引擎只是任意的 JS 代码按需执行的环境，是其他线程调用触发 JS 引擎执行 JS 代码，比如：一个按钮点击触发事件，接着调用 js 引擎执行等

JS 引擎工作流程图如下：

![20230117-3](https://github.com/peng-yin/note/assets/25680922/d906c46f-fcbc-4eec-84ae-11effa21b945)

### React 生命周期

为了更好了解 React Fiber 架构，我们需要对比 React15 和 React16 的生命周期，具体如下：

#### React15 的生命周期

在 15 版本的时候，一个完整的组件生命周期如下（按照执行顺序）：

- constructor()，组件的构造函数，用来初始化 state
- componentWillMount()，初始化渲染前时调用
- componentDidMount()，初始化渲染后调用
- componentWillReceiveProps()，父组件修改组件的 props 时会调用
- render()，每次渲染时候会调用
- componentWillUpdate()，组件更新前调用
- shouldComponentUpdate()，组件更新时调用，主要判断组件要不要更新
- componentDidUpdate()，组件更新后调用
- componentWillUnmount()，组件卸载时调用

![20230117-4](https://github.com/peng-yin/note/assets/25680922/e1420164-c45b-4a79-8c2a-1386e533917c)

按照不同时期，执行过程是不一样，具体可以见 React 的生命周期更改相关文章。

#### React16 生命周期

相比较 React15，16 版本基于 Fiber 架构主要对更新周期的函数做了调整，整个生命周期如下：

- constructor()，组件的构造函数，用来初始化 state
- getDerivedStateFromProps()，初始化/更新时调用，使用 props 来派生/更新 state。
- componentDidMount()，初始化渲染后调用
- shouldComponentUpdate()，
- render()，每次渲染时候会调用
- shouldComponentUpdate()，组件更新时调用，主要判断组件要不要更新
- getSnapshotBeforeUpdate()，返回值会作为第三个参数给到 componentDidUpdate。它的执行时机是在 render 方法之后，真实 DOM 更新之前。可以同时获取到更新前的真实 DOM 和更新前后的 state&props 信息。
- componentDidUpdate()，组件更新后调用，从 getSnapshotBeforeUpdate 获取到的值
- componentWillUnmount()，组件卸载时调用

对比一下，React 16 废弃的是哪些生命周期：

- componentWillMount；
- componentWillUpdate；
- componentWillReceiveProps

这些生命周期的共性，就是它们都处于 render 阶段，都可能重复被执行，而且由于这些 API 常年被滥用，它们在重复执行的过程中都存在着不可小觑的风险。

为什么废弃这些生命周期，因为引用了 Fiber 架构，render 阶段是允许暂停、终止和重启的。这就导致 render 阶段的生命周期都是有可能被重复执行的。

React16 生命周期图如下：

![20230117-5](https://github.com/peng-yin/note/assets/25680922/f3e8328c-9e5c-43bc-bbea-ee9d44583d43)

### React 虚拟 DOM

> 虚拟 DOM（Virtual DOM）本质上是 JS 和 DOM 之间的一个映射缓存，它在形态上表现为一个能够描述 DOM 结构及其属性信息的 JS 对象。

记住两个点：

- 虚拟 DOM 是 JS 对象
- 虚拟 DOM 是对真实 DOM 的描述

虚拟 DOM 出现 react 生命周期的两个节点：

1. 挂载阶段，React 将结合 JSX 的描述，构建出虚拟 DOM 树，然后通过 ReactDOM.render 实现虚拟 DOM 到真实 DOM 的映射
2. 更新阶段，页面的变化在作用于真实 DOM 之前，会先作用于虚拟 DOM，虚拟 DOM 将在 JS 层借助算法先对比出具体有哪些真实 DOM 需要被改变，然后再将这些改变作用于真实 DOM，这里就需要 DOM Diff 算法。

为什么需要虚拟 DOM？并不是因为虚拟 DOM 有更高的性能，而是因为虚拟 DOM 的优越之处在于，它能够在提供更爽、更高效的研发模式（也就是函数式的 UI 编程方式）的同时，仍然保持一个还不错的性能。解决了以下问题：

1. 研发体验/研发效率的问题，解决以往模板和数据，需要重复调整的问题
2. 跨平台的问题，从 web、小程序、app 等，一套虚拟 DOM，结合不同渲染逻辑，满足各类跨端场景

而在虚拟 DOM 这一块，Fiber 架构的引入，最大的调整就是虚拟 DOM 更新中的 diff 算法，由于分片渲染，不需要一次将 diff 执行，可以分批计算从而减少 diff 算法的复杂度。

### Stack Reconciler(栈协调器)

在了解 Fiber 架构之前，需要对 React15 的 Stack Reconciler(栈协调器) 做一次完整了解。先了解一下什么 Reconciler 协调器，在 React 中是这么定义的：

> Virtual DOM 是一种编程概念。在这个概念里，UI 以一种理想化的，或者说“虚拟的”表现形式被保存于内存中，并通过如 ReactDOM 等类库使之与“真实的” DOM 同步。这一过程叫作 Reconciler 协调（调和）。

所以实现 Reconciler，其实就是实现虚拟 DOM 到真实 DOM 渲染的整个逻辑过程，因此调和 !== Diff，但是 Diff 确实是调和过程中最具代表性的一环。

那么要了解 React15 是如何实现 Stack Reconciler，最重要的两块：

- Diff 算法策略
- 找到 diff 节点并 `同步` 更新渲染

Diff 算法策略要点：(主要是树递归)

1. Diff 算法性能突破的关键点在于“分层对比”；
2. 类型一致的节点才有继续 Diff 的必要性；
3. key 属性的设置，可以帮我们尽可能重用同一层级内的节点。

## Fiber 架构

我们先来看看 React  团队在“React 哲学”中对 React 的定位：

> 我们认为，React 是用 JavaScript 构建快速响应的大型 Web 应用程序的首选方式。它在 Facebook 和 Instagram 上表现优秀。

`快速响应` 是 React 哲学理念，因此 Fiber 架构的出现是为了让 React 框架能更加快速响应用户的操作。

### 是什么

什么是 Fiber？从字面上来理解，Fiber 这个单词翻译过来是“丝、纤维”的意思，是比线还要细的东西。在计算机科学里，我们有进程、线程之分，而 Fiber 就是比线程还要纤细的一个过程，也就是所谓的“纤程”。纤程的出现，意在对渲染过程实现更加精细的控制。

Fiber 的概念理解：

- 从架构角度来看，Fiber 是对 React 核心算法（即调和过程）的重写
- 从编码角度来看，Fiber 是 React 内部所定义的一种数据结构，它是 Fiber 树结构的节点单位，也就是 React 16 新架构下的“虚拟 DOM”；
- 从工作流的角度来看，Fiber 节点保存了组件需要更新的状态和副作用，一个 Fiber 同时也对应着一个工作单元。

从架构角度理解 Fiber:

- 架构核心：“可中断”“可恢复”与“优先级”
- 可中断，指的是在 Fiber 架构下，任何工作任务都可以被更高优先级的任务中断
- 可恢复，指的是被中断的任务可以被恢复继续执行
- 优先级，指的是每个任务都有自己的优先级定义
- 因此需要增加“Scheduler（调度器）”，作用是调度更新的优先级的任务

### 怎么解决问题

有了 Fiber 架构，怎么解决 React15 所面临的问题，虚拟 DOM 同步渲染真实 DOM 导致页面卡顿？

- 将虚拟 DOM，从原有的树结构，改为链表结构，拆分成一个个 Fiber 树节点
- 利用 Fiber 架构，将渲染过程拆分成一个个工作单元任务，设置优先级，支持可中断、可恢复
- 这样子当需要渲染复杂 DOM 时候，同时不影响其他优先级较高工作任务执行

可以参考下图，了解一下 Fiber 架构工作图：

![20230117-2](https://github.com/peng-yin/note/assets/25680922/5ff8a2de-739a-48c4-b990-07342a065f9b)

当然这样子讲只是简单的原理，还需要弄明白异步后可能产生更多问题？比如如何定制优先级，当两个同样优先级的任务相遇的时候如何解决，这些放在第二章讲解。

## 总结

经过第一篇 Fiber 文章学习，大概了解到 Fiber 架构出现的背景和原因，以及它是什么，是如何工作的解决之前所遇到问题。简单总结一下：

- React 中定义 Reconciler 协调，指的是将虚拟 DOM 渲染到真实 DOM 的过程，React15 之前采用是 stack Reconciler 栈协调，同步渲染机制导致页面卡顿
- React16 之后采用 Fiber Reconciler，实现异步渲染 DOM
- 采用新的 Fiber 架构，同时影响到 React 的整个生命周期，主要是在更新阶段的生命周期

后续深入了解请看第二篇章《从 React 中学习 Fiber 架构 (二)》。

## 额外话题（Vue.js 对比）

相比较 React 做 Fiber 架构优化，主要是针对事件做了时间分片，那么为什么 Vue3(Vue@next) 版本并不需要做呢？Vue.js 作者尤雨溪是这样子回答的：

> 尤雨溪：在 Web 应用中，「可中断式更新」主要是由大量 CPU 计算加上复杂 DOM 操作引起的。时间分片旨在让应用在 CPU 进行大量计算时也能与用户交互，但时间分片只能对大量 CPU 计算进行优化，无法优化复杂 DOM 操作，因为要确保用户正在操作的界面是最新的状态才行。因此，我们可以考虑两种不同的可中断式更新的场景：
> 1. CPU 计算量不大，但 DOM 操作非常复杂（比如说你向页面中插入了十万个节点）。这种场景下不管你做不做时间分片，页面都会很卡。
> 2. CPU 计算量非常大。理论上时间分片在这种场景里会有较大收益，但是人机交互研究表明，除了动画之外，大部分用户不会觉得 10 毫秒和 100 毫秒有很大区别。
> 也就是说，时间分片只在 CPU 需要连续计算 100 毫秒以上的情况下才有较大收益。有意思的地方就出现了，在 React 经常会出现 100 毫秒以上的计算量，因为
> 3. Fiber 架构的复杂性导致 React 的虚拟 DOM 协调效率较低，这是系统性的问题。
> 4. React 使用 JSX 导致它的渲染效率比 template 低，因为 template 很容易做静态分析和优化。
> 5. React Hooks 将大部分组件树的优化 API 暴露给开发者，开发者很多时候需要手动调用 useMemo 来优化渲染效率。这意味着 React 应用默认就有 render 过多的问题。更严重的是，这些优化在 React 里很难自动化。
> 6. 这些优化要求开发者正确设置依赖数组
> 7. 盲目添加 useMemo 会导致应该 render 的没 render。
> 很不幸，大部分开发者都很懒，不会在每个地方都加上优化，因此大部分 React 应用都会有大量的没必要的 CPU 计算工作。
> 对比较而言，Vue 解决了上述问题：
> 8. Vue 的架构里没有时间分片，也就没有 Fiber，因此简单了很多，这使得渲染可以更快。
> 9. Vue 通过分析 template、简化协调过程，做了大量的 AOT 优化，性能测试结果表明大部分的 DOM 内容有 80% 属于静态内容，因此 Vue 3 的协调速度比 Svelte 快，花费的时间比 React 的 1/10 还少。
> 10. 通过数据响应式追踪，Vue 可以做到组件树级别的优化，比如把插槽编译为函数以避免 children 的变化引发 re-render，比如自动缓存内联事件处理函数以避免 re-render。Vue 3 可以做到在不借助开发者的任何手动优化的情况下，防止子组件在非必要的情况下 re-render。这意味着同样一次更新，React 应用可能要 re-render 多个组件，而 Vue 应用很可能只 re-render 一个组件。
> 因此，在默认情况下，Vue 3 应用会比 React 应用少花费很多 CPU 时间，因而遇到 CPU 连续计算时间超过 100 毫秒的机会相当少，除非是极端情况。但大部分极端情况是 DOM 操作过于复杂，而不是 CPU 计算量太大。

进行汇总一下描述，Vue3 之所以没有使用 Fiber 架构，主要有以下几个原因：

1. Vue.js 针对 template 渲染机制做了多重优化，包括 AOT 优化 (在构建的时候提前进行编译，提前将 template 转义成 render 函数) 等，使得 DOM 元素渲染更快
2. 复杂 DOM 渲染出现超过 100ms 以上的计算，是因为 React 本身机制导致，并不是所有复杂的 DOM 渲染都会需要 100ms
3. React Hook 的暴露增加渲染效率的复杂度，从而导致 React 渲染更慢，从而需要 Fiber 架构去协调
4. Vue 数据响应式追踪机制，避免了多次重复 render 组件树，提高渲染效率
5. Vue 使用 Fiber 架构去实现，确实可以有好处，但是会增加整体代码体积和复杂度，投入产出比太低

### 基础概念

#### AOT vs JIT

- AOT，Ahead Of Time，提前编译或预编译，宿主环境获得的是编译后的代码，在浏览器中我们可以直接下载并运行编译后的代码，比如：Vue 的 template 是通过 Vue-loader 编译后才能使用。
- JIT，Just In Time，即时编译 ，代码在宿主环境编译并执行，每个文件都是单独编译的，当我们更改代码时不需要再次构建整个项目，比如：React 中 JSX 只有在浏览器运行的时候才知道具体代码。
