---
title: 动态执行 JS
date: 2024-11-25T10:03:10+08:00
updated: 2024-11-25T10:03:18+08:00
dg-publish: false
---

传入一个字符串当成 JS 语句进行执行

例如

```js
exec('console.log(123)')
```

实现

1. `eval(code)` 把代码传进去，同步代码，执行的时候作用域为当前作用域
2. `setTimeout(code, 0)` 第一个参数为字符串的时候会当成代码执行，异步，全局作用域
3. 创建 `script` 元素，设置内容，同步，全局作用域
4. `new Function(code)` 同步，全局作用域

```js
var a = 1
function exec(code) {
  var a = 2
  // eval(code)  // a 2    sync
  // setTimeout(code, 0)  // sync    a 1  
  /*
  const script = document.createElement('script')
  script.innerHTML = code
  document.body.appendChild(script)  
  // a 1    sync
  */
  const fn = new Function(code)
  fn()  // a 1    sync
}
exec('console.log("a", a)')
console.log('sync')
```
