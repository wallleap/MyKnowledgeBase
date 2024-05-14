---
title: 061-Node.js 技术架构
date: 2024-04-03 10:58
updated: 2024-04-30 14:55
---

Node.js 版本

- 双数是稳定版，单数非稳定版
- 使用 8 以上的版本

安装

- Windows 用户
	- 去官网下载
	- 可以考虑下载 nvm 之类的版本管理
- mac 用户
	- 使用 Homebrew 安装
	- `brew install nvm` 安装版本管理

周边工具

- nrm 用于切换下载源
- pnpm/yarn 和 yrm 代替
- ts-node 可以运行 TypeScript 的 node

Node.js 不是

- web 框架
	- 并不是 web 后端框架
	- 所以不能和 Flask 或 Spring 对比
- 编程语言
	- 并不是后端的 JS
	- 不能与 Python 或 PHP 对比

Node.js 是

- 一个平台
	- 将多种技术组合在一起
	- 让 JS 也能调用系统接口、开发后端应用

Node.js 用到了哪些技术

- V8 引擎
- libuv
- C/C++ 实现的 c-ares、http-parser、OpenSSL、zlib 等库

## Node.js 技术架构

![](https://cdn.wallleap.cn/img/pic/illustration/202404031125748.png)

最上层暴露的 API，依赖中层，中层依赖底层

随着版本更新，其架构也在一直变化中

如果要看源代码，推荐看 0.10 版本，因为这一版使用了很长一段时间，而且源代码比最新版少很多

要了解更多，可以看 yjhjstz/deep-into-node

##  Node.js bingdings

C/C++ 实现了一个 http_parser 库，很高效，想用 JS 调用这个库，就需要一个中间桥梁

1. Node.js 用 C++ 对 http_parser 进行封装，让它符合某些要求，封装的文件叫做 http_parser_bindings.cpp
2. 用 Node.js 提供的编译工具将其编译为 `.node` 文件（或其他方式）
3. JS 代码可以直接 require 这 `.node` 文件
4. 这样 JS 就能调用 C++ 库，中间桥梁就是 binding
5. 由于 Node.js 提供了很多 binding， 所以加个 s，这就是 bingdings

### JS 如何调用 C++

一个 `.cc` 文件，编译后可以直接在 JS 中 `require` 并使用传入的函数

```js
// test.js
const addon = require('./build/Release/addon');

console.log('This should be eight:', addon.add(3, 5));
```

### C++ 回调 JS

将 JavaScript 函数传给 C++ 函数并从那里执行它们

```js
// test.js
const addon = require('./build/Release/addon');

addon((msg) => {
  console.log(msg);
// Prints: 'hello world'
});
```

Node.js v0.10 的 deps（依赖）目录

## V8

功能：

- 将 JS 源代码变成本地代码并执行
- 维护调用栈，确保 JS 函数的执行顺序
- 内存管理，为所有对象分配内存
- 垃圾回收，重复利用无用的内存
- 实现 JS 的标准库

注意：

- V8 不提供 DOM API
- V8 执行 JS 是单线程的，可以开启两个线程分别执行 JS
- V8 本身是包含多个线程的，如垃圾回收为单独线程
- 自带 event loop 但 Node.js 基于 libuv 自己做了一个

## Event Loop

[Event Loop、计时器、nextTick - 掘金 (juejin.cn)](https://juejin.cn/post/6844903582538399752)

什么是 Event（事件）

- 计时器到期了
- 文件可以读取了、读取出错了
- socket 有内容了、关闭了

什么是 Loop

- 循环，比如 `while(true)` 循环
- 由于事件是分优先级的，所以处理起来也是分先后的
	- 举例：三种不同的事件 `setTimeout(f1, 100)`、`fs.readFile('/1.txt', f2)`、`server.on('close', f3)` 如果同时触发，Node 会怎么办
	- 人为规定顺序（优先级），并且轮询
- Node.js 需要按顺序轮询每种事件，这种轮询往往都是循环的

Event Loop

- 操作系统可以触发事件，JS 可以处理事件，Event Loop 就是对事件处理顺序的管理
- 重点阶段
	- **timers** 检查计时器
	- **poll** 轮询，检查系统事件
	- check 检查 **setImmediate** 回调
	- 其他阶段用得比较少
- 大部分时间 Node.js 都停在 poll 轮询阶段，大部分事件都在 poll 阶段被处理，如文件、网络请求

![](https://cdn.wallleap.cn/img/pic/illustration/202404301418097.png)

举例来说，你设置了一个计时器在 100 毫秒后执行，然后你的脚本用了 95 毫秒来异步读取了一个文件：

```js
const fs = require('fs');

function someAsyncOperation(callback) {
  // 假设读取这个文件一共花费 95 毫秒
  fs.readFile('/path/to/file', callback);
}

const timeoutScheduled = Date.now();

setTimeout(() => {
  const delay = Date.now() - timeoutScheduled;

  console.log(`${delay}毫秒后执行了 setTimeout 的回调`);
}, 100);

// 执行一个耗时 95 毫秒的异步操作
someAsyncOperation(() => {
  const startCallback = Date.now();

  // 执行一个耗时 10 毫秒的同步操作
  while (Date.now() - startCallback < 10) {
    // 什么也不做
  }
});
```

当 event loop 进入 poll 阶段，发现 poll 队列为空（因为文件还没读完），event loop 检查了一下最近的计时器，大概还有 100 毫秒时间，于是 event loop 决定这段时间就停在 poll 阶段。在 poll 阶段停了 95 毫秒之后，fs.readFile 操作完成，一个耗时 10 毫秒的回调函数被系统放入 poll 队列，于是 event loop 执行了这个回调函数。执行完毕后，poll 队列为空，于是 event loop 去看了一眼最近的计时器（译注：event loop 发现卧槽，已经超时 95 + 10 - 100 = 5 毫秒了），于是经由 check 阶段、close callbacks 阶段绕回到 timers 阶段，执行 timers 队列里的那个回调函数。这个例子中，100 毫秒的计时器实际上是在 105 毫秒后才执行的。

注意：为了防止 poll 阶段占用了 event loop 的所有时间，libuv（Node.js 用来实现 event loop 和所有异步行为的 C 语言写成的库）对 poll 阶段的最长停留时间做出了限制，具体时间因操作系统而异。

## libuv

各操作系统上有各自的异步 I/O，Ryan 为了实现一个跨平台的异步 I/O 库，开始写 libuv

libuv 会根据系统自动选择合适的方案

功能：可以用于 TCP/UDP/DNS/文件等的异步操作

## 总结

- 用 libuv 进行异步 I/O 操作
- 用 event loop 管理事件处理顺序
- 用 C/C++ 库高效处理 DNS/HTTP..
- 用 bindings 让 JS 能和 C/C++ 沟通
- 用 V8 运行 JS
- 用 Node.js 标准库简化 JS 代码

这就是 Node.js

## Node.js 工作流程

![](https://cdn.wallleap.cn/img/pic/illustration/202404301437030.png)

## Node.js API

官方提供的函数

文档地址：

- nodejs.org/api/
- nodejs.cn/api
- <https://devdocs.io/>

有哪些 API

```js
Assertion 断言               Path *
Testing                     Performance Hooks
Async Hooks                 Process *
Buffer * 一小段缓存           Query Strings *
Child Processes *           Readline
Cluster *                   REPL
Console                     Report
Crypto                      Stream *
Debugger *                  String Decoder
DNS                         Timers *
Errors                      TLS/SSL
Events *                    Trace Events
File System *               TTY
Globals *                   UDP/Datagram
HTTP *                      URL *
HTTP/2                      Utilities
HTTPS                       V8
Inspector                   VM
i18n                        Worker Threads *
Net                         Zlib
OS
```
