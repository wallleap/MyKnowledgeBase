---
title: 使用立即执行函数 IIFE 提升性能
date: 2024-10-14T02:41:24+08:00
updated: 2024-10-14T02:54:18+08:00
dg-publish: false
---

有些不需要每次都判断，一开始就能确定的

```js
const addEvent = (function() {
  if (1) {
    return function () {}
  } else if (2) {
    return function () {}
  } else if (3) {
    return function () {}
  } else {
    return function () {}
  }
})()
addEvent()
```

例如

```js
const request = (() => {
  if (typeof window !== 'undefined') {
    return (options) => {
      // 浏览器端的 ajax
    }
  } else {
    return (options) => {
      // node 的 http
    }
  }
})()
// 之后调用 request 的时候就已经确定是哪种的了
```

正则表达式，字符串等都会占用内存，会让内存忽上忽下的，可以使用工厂函数

```js
const createRemoveSpace = () => {
  const reg = /\s/g,  // 单独提出为变量会一直占用内存
    replacement = ''
  return str => str.replace(reg, replacement)
}

const createSpace = createRemoveSpace()  // 内存升高
createSpace('111 11 11')  // 使用的时候由于闭包不会忽上忽下
createSpace = null // 不需要的时候回收
```
