---
title: DomParser 解析 DOM 树
date: 2024-10-24T01:16:58+08:00
updated: 2024-10-24T01:22:03+08:00
dg-publish: false
---

浏览器环境

```js
function removeTag(fragment) {
  return (
    new DOMParser().parseFromString(fragment, 'text/html').body.textContent || ''
  )
}

console.log(removeTag('<div>123</div>'))  // 123
```

原理

```js
const str = '<div>123</div>'
const parser = new DOMParser()
const doc = parser.parseFromString(str, 'text/html')  // 第二个参数是 MIME 类型
console.log(doc)
console.log(doc.querySelector('div'))
console.log(doc.body.textContent)
```
