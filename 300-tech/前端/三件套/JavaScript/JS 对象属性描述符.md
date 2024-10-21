---
title: JS 对象属性描述符
date: 2024-09-07T11:17:57+08:00
updated: 2024-09-12T07:28:11+08:00
dg-publish: false
---

## 可被遍历

```js
const obj = {
  a: 1,
  b: 2
}
Object.prototype.c = function() {
  console.log(c)
}
for (let key in obj) {
  console.log(key)
}
console.log(obj.hasOwnProperty('a'))  // true
console.log(obj.hasOwnProperty('c'))  // false
```

发现打印出了 a b c

但是其他原型上的属性却没被打印出来

属性描述符

```js
const desc = Object.getOwnPropertyDescriptor(Object.prototype, 'hasOwnProperty')
console.log(desc)
/*
{
  value: [Function: hasOwnProperty],
  writable: true,
  enumerable: false,
  configurable: true
}
*/
```

现在想让 c 不可遍历就需要用另一种方式定义

```js
Object.defineProperty(Object.prototype, 'c', {
  value: function() {
    console.log(c)
  },
  writable: true,
  enumerable: false,
  configurable: true
})
```

这样再遍历就只输出 a b 了

## 访问器

```js
const o = (function () {
  const obj = {
    a: 1,
    b: 2
  }
  return {
    get: function (k) {
      return obj[k]
    }
  }
})()
console.log(o.get('a'))
```

不修改上述代码，修改 obj 对象

1、考虑使用 valueOf

```js
console.log(o.get('valueOf'))
```

报了类型错误 `TypeError: Cannot convert undefined or null to object`，`this` 指向是全局，`(Object.prototype.valueOf)()`

如果 get 中返回的是直接调用了的就可以

```js
const o = (function () {
  const obj = {
    a: 1,
    b: 2
  }
  return {
    get: function (k) {
      return obj[k]()
    }
  }
})()
console.log(o.get('valueOf'))
```

2、读这个属性的时候直接就是一个函数调用

当这个属性是一个访问器的时候

```js
Object.defineProperty(Object.prototype, 'abc', {
  get: function () {  // 只要一访问 abc 这个属性就相当于运行这个函数
    return this
  }
})
console.log(o.get('abc'))  // obj 本身

const obj2 = o.get('abc')
obj2.c = 3
obj2.a = 4
console.log(o.get('c'))
console.log(o.get('a'))
```

防止：

第一种 只能读自身的属性

```js
const o = (function () {
  const obj = {
    a: 1,
    b: 2
  }
  return {
    get: function (k) {
      if (obj.hasOwnProperty(k))
        return obj[k]
    }
  }
})()
```

第二种 设置原型为空

```js
const o = (function () {
  const obj = {
    a: 1,
    b: 2
  }
  Object.setPrototypeOf(obj, null)
  return {
    get: function (k) {
      return obj[k]
    }
  }
})()
```
