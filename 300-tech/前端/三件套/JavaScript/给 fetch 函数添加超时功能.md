---
title: 给 fetch 函数添加超时功能
date: 2024-10-16T05:24:15+08:00
updated: 2024-10-16T05:24:29+08:00
dg-publish: false
---

AJAX 有两种实现方式 XHR 和 fetch

XHR 是自带超时功能的，而 fetch 没有

## 最简单的实现

```js
const controller = new AbortController()
fetch(url, {
  signal: controller.signal
})
setTimeout(() => {
  controller.abort()
}, 100)
```

不能通用

## 封装

如果直接放一个函数里面，那么就变成封装 AJAX 了，需要传很多参数

```js
function request(timeout) {
  // ...
}
```

mockjs 这个库可以拦截 AJAX 请求，原理就是先备份原来的 XHR，然后直接修改 XHR 这个函数，但是这样有**侵入性**

没有侵入性，保持通用性

```js
function createFetch(timeout) {
  return (resource, options) => {
    let controller = new AbortController()
    options ||= {}
    options.signal = controller.signal
    setTimeout(() => {
      constroller.abort()
    }, timeout)
    return fetch(resource, options)
  }
}

// 不需要超时，直接用原来的 fetch
fetch()
// 需要超时
const toutFetch = createFetch(3000)
toutFetch()
// createFetch(3000)(url)
```
