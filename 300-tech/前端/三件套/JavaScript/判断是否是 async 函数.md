---
title: 判断是否是 async 函数
date: 2024-10-14T04:03:06+08:00
updated: 2024-10-14T04:03:18+08:00
dg-publish: false
---

```js
function isAsyncFunction(func) {
  return typeof func === 'function' && func[Symbol.toStringTag] === 'AsyncFunction'
}
console.log(isAsyncFunction(function() {}))
console.log(isAsyncFunction(async function() {}))
console.log(isAsyncFunction(async () => {}))
```
