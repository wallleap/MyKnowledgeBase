---
title: console 对象
date: 2024-09-07T01:56:43+08:00
updated: 2024-09-07T01:56:50+08:00
dg-publish: false
---

```js
// 1. 打印调试信息
console.debug('debug')  // level 需要勾选 Verbose

// 2. 打印消息
// 2.1 普通消息
console.log('log')

// 2.2 信息
console.info('info')

// 2.3 表格
console.table([
  {
    name: 'John',
    age: 18
  },
  {
    name: 'Jane',
    age: 20
  }
])

// 2.4 分组
const label = 'group11111'
console.group(label)
console.groupCollapsed(label)
console.log('log')
console.log('more log')
console.groupEnd(label)

// 2.5 对象结构
console.dir(document.body)

// 2.6 计时
console.time('time')
const start = Date.now()
while (Date.now() - start < 1000) {}
console.timeEnd('time')

// 2.7 计数
const start1 = Date.now()
while (Date.now() - start1 < 1) {
  console.count('loop')
}
console.countReset('loop')

// 2.8 堆栈
function b() {
  console.trace('trace')
}
function a() {
  b()
}
a()

// 2.9 断言
function sum(a, b) {
  return a + b
}
console.assert(sum(1, 2) === 3)
console.assert(sum(1, 2) !== 3)

// 3. 打印警告
console.warn('warn')

// 4. 打印错误
console.error('error')

// 5. 清空消息
console.clear()

// 6. 给消息添加样式
const style = `
  color: blue;
  background-color: yellow;
  font-size: 20px;
  font-weight: bold;
  padding: 5px;
  border-radius: 5px;
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.5);
  text-shadow: 0 0 5px rgba(0, 0, 0, 0.5);
`
console.log('%c red', style)

```

