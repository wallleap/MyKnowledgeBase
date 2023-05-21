---
title: 014-Symbol、Set、Map、WeakMap
date: 2023-03-10 20:38
updated: 2023-03-19 22:34
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - 014-Symbol、Set、Map、WeakMap
rating: 1
tags:
  - JavaScript
category: web
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
description: 文章描述
source: null
url: //myblog.wallleap.cn/post/1
---

## Symbol

- JS 的内置原始类型
- 每个 Symbol 都**不相同**，独一无二的
- 可以用在对象的属性中，例如一般情况下 `a: 1` 和 `"a": 1` 是一样的，对象作为属性的时候也会转成字符串，如 `{}: 1` 是 `"": 1`，`{a: 1}:1` 是 `[object Object]:1`；但是用 Symbol 就可以直接作为标记使用，每个标记都是独一无二的

```js
const sym1 = Symbol()
const sym2 = Symbol('foo')
const sym3 = Symbol('foo')
console.log(sym2 === sym3)  // false
let obj = {[sym1]: 'a'}
obj[sym2] = 'b'
console.log(obj)
console.log(obj[sym1])  // 'a'
```

## Set

Set 对象是值的**集合**，可以按照插入的顺序迭代它的元素；Set 中的元素是唯一的（没有重复项）

用法：

- `let set = new Set()`  创建一个 Set，默认有一个 size 属性，size 为 0 表示为空；可以传递字串和数组
- `set.add(val)`  增
- `set.delete(val)`  删
- `set.has(val)`  是否有
- `set.size`  长度
- `set.clear()`  清空
- `set.forEach(callback)`  遍历
- `[...set]`  转为数组

可以使用 Set 实现数组去重

```js
[...new Set(arr)]
```

用 Set 在原数组上操作

```js
const remove = arr => {
  let set = new Set()
  for(let i=0; i<arr.length; i++) {
    if(!set.has(arr[i])) {  // set 中没有这项就添加
      set.add(arr[i])
    } else {  // 有则删掉这个，并让下标 -1
      arr.splice(i, 1)
      i--
    }
  }
}
const arr1 = [1, '2', 3, 2, 'a', 'a']
remove(arr1)
console.log(arr1)
```

## Map

Map 对象是键值对的集合。Map 中的一个键只能出现一次，任何值都可以作为键和值

Object 和 Map 的区别

- Map 键可以是任何类型，Object 键只能是字符串和 Symbol
- Map 的键是有序的，Object 是无序的
- Map 可以获取 size，Object 需要自己计算
- Map 性能更好
- Map 是可迭代的，可以用 for-of；Obeject 是不可迭代的，可用 for-in 遍历

```js
let map = new Map()
map.set('name', 'luwang')
map.set(1, 100)
let obj = {a: 1}
map.set(obj, 2)
console.log(map.get('name'))
console.log(map.get(obj))
map.delete('name')
map.forEach((val, key) => {
  console.log(val, key)
})
for(let [key, value] of map) {
  console.log(key, value)
}
console.log(map.size)
```

## WeakMap

WeakMap 对象是一组键/值对的集合，其中的键是弱引用的。其键必须是对象，而值可以是任意的。用法和 Map 基本一致，不过 WeakMap 是不可遍历的

如果 key 是引用类型，推荐使用 WeakMap（例如 DOM 对象映射）

```js
let wk = new WeakMap()
const o1 = {},
	  o2 = function() {},
	  o3 = []
wm.set(o1, 1)
wm.set(o2, o3)
console.log(wm.get(o1))
console.log(wm.get(o2))
```
