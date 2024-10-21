---
title: JS 中可迭代
date: 2024-09-13T02:46:59+08:00
updated: 2024-09-13T02:47:07+08:00
dg-publish: false
---

```js
Object.prototype[Symbol.iterator] = function() {
  return Object.values(this)[Symbol.iterator]()
}
const [a, b] = { a: 1, b: 2 }
console.log(a, b)
```
