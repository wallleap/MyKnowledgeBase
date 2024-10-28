---
title: Node.js 事件循环机制：解析 JavaScript 异步魔力的核心
date: 2023-08-03T03:47:00+08:00
updated: 2024-08-21T10:32:30+08:00
dg-publish: false
aliases:
  - Node.js 事件循环机制：解析 JavaScript 异步魔力的核心
author: Luwang
category: web
comments: true
cover: //cdn.wallleap.cn/img/post/1.jpg
description: 文章描述
image-auto-upload: true
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
rating: 1
source: #
tags:
  - Node
  - web
url: //myblog.wallleap.cn/post/1
---

## 前言

我们在做项目开发时，可以用 `Node.js` 写后端拿到后端的数据。这就使得我们能快速且较低的成本过渡成为一名全栈工程师，当然你也要会数据库。我们知道 `Node.js` 是可以运行 `JavaScript` 代码的，`JavaScript` 中是有异步代码的，那它是怎样处理异步的呢？ `Node.js` 它的核心思想是通过事件驱动来处理异步操作的，而且它和浏览器是不一样的，接下来我们一起来看一下吧。

## 什么是 Node?

`Node.js` 是一个开源的、跨平台的 `JavaScript` 运行时环境，基于 `Chrome V8` 引擎构建而成。它允许开发人员使用 `JavaScript` 编写服务器端和命令行工具，实现高性能和可伸缩的网络应用程序。而且它是非阻塞的、事件驱动的 `I/O` 模型，有着自己独特的一套事件循环机制可以处理 `JavaScript` 中的异步。

### 运行在单线程上

- 为了处理异步，线程必须要有循环，不断地检查是否有未处理的事件，依次处理。
- 吞吐量。
- 如果同时有多个客户端访问 `node`,`node` 仅仅有一个线程，不会给每一个客户都创建一个线程。

> 这样的好处是：操作系统完全没有创建线程和销毁线程的时间开销。

> 弊端是：
>
> 1.  无法利用多核 CPU。
> 2.  错误会引起整个应用无法继续调用异步 I/O。
> 3.  大量计算占用 CPU 导致无法继续调用异步 I/O。

### libuv 库

看下图： ![](https://cdn.wallleap.cn/img/pic/illustration/202308031548582.png) `Node` 中专门有个 `libuv` 库为 `Node.js` 提供了事件驱动的底层基础。`Node.js` 的事件驱动和异步 `I/O` 功能正是基于 `libuv` 实现的。

### 应用场景

由于 `Node.js` 善于 `I/O` (任务调度) 不善于计算，有以下应用场景：

1. 考试系统，实时交互系统等高并发的 `web` 应用。
2. 多人联网的游戏。
3. 聊天室，图文直播。

## Node 事件循环机制

### 是什么？

`Node.js` 的事件循环机制是指用于处理异步操作和事件驱动的核心机制。`Node.js` 采用单线程的事件驱动模型，通过事件循环来处理和调度异步操作，以实现高效的非阻塞 `I/O`。

### 基本工作原理

`Node.js` 的事件循环机制分为本轮循环和次轮循环，像 `process.nextTick()` 和 `Promise.then` 的回调等异步任务追加到本轮循环；

`Promise.then` 的回调，**微任务队列追加在 `process.nextTick()` 后面**，也属于本轮循环。

`setTimeout`,`setInterval`,`setImmediate` 的回调等异步任务追加到次轮循环。

我们直接看下面展示：

```
  ┌───────────────────────────┐
┌─>│           timers          │  //此阶段包括setTimeout,setInterval
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │     pending callbacks     │  //大部分的普通回调事件
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │
│  └─────────────┬─────────────┘      ┌───────────────┐
│  ┌─────────────┴─────────────┐      │   incoming:   │
│  │           poll            │<─────┤  connections, │
│  └─────────────┬─────────────┘      │   data, etc.  │
│  ┌─────────────┴─────────────┐      └───────────────┘
│  │           check           │  //setimmediate 调用回调
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
└──┤      close callbacks      │  //关闭回调
   └───────────────────────────┘
```

上面展示了 `Node.js` 的事件循环的各个阶段，为了搞清楚执行顺序，我们来看一个问题：

> 两个异步任务开启，是谁快就谁先完成，还是有先后执行顺序？

异步队列有多种，队列之间有先后执行顺序，在某个队列内部，是谁快就谁先完成。注意是队列之间有先后执行顺序队列内部谁快谁先执行。

还是有点难理解？详细描述一下：

`Node.js` 开始执行代码时就会进行事件循环的初始化，如果你有一份代码需要放在 `Node` 里面执行，在执行之前就会进行一个事件循环的初始化，但是这个时候事件循环还没有开始，`Node` 会先去完成一些任务，规划好这些任务：同步任务，发出异步请求，规划定时器生效的时间，执行 `nextTick` 等等；最后事件规划好了后才正式开始执行代码。浏览器里面代码执行前需要编译，而 `Node` 是进行规划同步和异步执行顺序。事件循环都是运行在单线程环境下的，我们说的高并发也是依靠事件循环，每产生一个事件就会加入到这个阶段对应的队列当中去，此时事件循环就会将这个队列当中的事件取出来使用。假设事件循环进入到了某个阶段，这个阶段其他事件队列里面也就准备就绪着，也会将当前的全部的回调方法执行完毕进入到下一个阶段。

### 一道题

以下代码在 `Node.js` 中输出顺序是什么：

```js
async function async1() {
    console.log('async1 start')
    await async2()
    console.log('async1 end')
}
async function async2() {
    console.log('async2')
}
console.log('script start')
setTimeout(function () {
    console.log('setTimeout0')
}, 0)
setTimeout(function () {
    console.log('setTimeout3')
}, 3)
setImmediate(() => console.log('setImmediate'))
process.nextTick(() => console.log('nextTick'))
async1()
new Promise(function (resolve) {
    console.log('promise1')
    resolve()
    console.log('promise2')
}).then(function () {
    console.log('promise3')
})
console.log('script end')
```

先执行同步代码：'script start'，'async1 start'，'async2'，'promise1'，'promise2'，'script end'；

接下来就是异步代码，'async1 end','promise3' 进入到微任务队列 quene mic\['async1 end','promise3'\]，'nextTick' 就入到 nextTick 队列 nextTickQueue\['nextTick'\] 属于本轮循环，而且 nextTick 队列比微任务队列先输出；

'setTimeout0''setTimeout3''setImmediate' 追加到次轮循环。'setTimeout3' 打印需要耗时所以先打印 'setImmediate' 最后再打印 'setTimeout3'

最终输出顺序为：

```
//script start
//async1 start
//async2
//promise1
//promise2
//script end
//nextTick
//async1 end
//promise3
//setTimeout0
//setImmediate
//setTimeout3
```

因为定时器是有误差的，所以有可能 'setTimeout3' 打印顺序在 'setTimeout0' 后面，'setImmediate' 最后打印。

## 小结

看到这里，相信大家对 `Node.js` 事件循环机制有了一定的了解。`Node.js` 的事件循环机制使得它能够高效地处理并发请求和异步操作，我们可以通过事件驱动的方式提高代码的可读性和可维护性，不得不说 `Node.js` 已经成为了我们前端在服务器端开发领域的重要工具之一。本篇文章到这就结束了，点赞与收藏是对作者最大的鼓舞ヾ (◍°∇°◍) ﾉﾞ
