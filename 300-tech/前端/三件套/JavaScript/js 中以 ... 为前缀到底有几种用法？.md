---
title: js 中以 ... 为前缀到底有几种用法？
date: 2023-08-04T06:03:00+08:00
updated: 2024-08-21T10:33:28+08:00
dg-publish: false
aliases:
  - js 中以 ... 为前缀到底有几种用法？
author: Luwang
category: web
comments: true
cover: //cdn.wallleap.cn/img/post/1.jpg
description: 文章描述
image-auto-upload: true
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
rating: 1
source: #
tags:
  - JS
  - web
url: //myblog.wallleap.cn/post/1
---

我报名参加金石计划 1 期挑战——瓜分 10 万奖池，这是我的第 2 篇文章，[点击查看活动详情](https://s.juejin.cn/ds/jooSN7t "https://s.juejin.cn/ds/jooSN7t") 。

ES6 开始，js 新增了剩余参数语法、展开语法等，它们有个共同之处就是都以 `...` 这么个符号为前缀，好像很多地方都可以用到，但实际上又不是同一回事，容易让初学者晕头转向。本篇就对目前笔者已知的 3 种关于 `...` 用法的总结归纳。

## 用法一：函数的剩余参数 (Rest parameters)

在**定义**函数的时候，**形参**当中，**最后一个**参数（`theArgs`）如果以 `...` 作为前缀，那么 `theArgs` 就会变为一个数组，调用该函数时，如果传入的实参数量大于等于定义函数时形参的数量，比如下面的例 1.1，形参定义了 3 个，却传入了 5 个实参，那么从 3 开始（包括），之后的实参都会被放入到 `theArgs` 中：

```js
// 例 1.1
function fn(a, b, ...theArgs) {
  console.log(theArgs)
}

fn(1, 2, 3, 4, 5) // [ 3, 4, 5 ]
```

请注意，`...theArgs` 必须作为最后一个形参，不然会报语法错误：“SyntaxError: Rest parameter must be last formal parameter”。 除箭头函数外，函数内都有个 [arguments](https://juejin.cn/post/7114202768285368350 "https://juejin.cn/post/7114202768285368350") 可以用来获取传递给函数的参数。那么它和剩余参数有什么区别呢？

1. 剩余参数，比如例 1.1 中的 `theArgs` 是个真数组，可以直接使用数组的方法，而 arguments 是一个类数组（array-like）对象，想要运用数组方法还得转成数组；
2. 剩余参数只包含那些没有对应形参的实参，比如例 1.1 中的 3、4 和 5，而 arguments 包含了传给函数的所有实参。
3. arguments 可以通过 callee 属性获取当前 arguments 所在的函数。

其实 arguments 是早期的规范中为了方便获取传入函数的参数而存在的，ES6 之后，我们应当尽量使用剩余参数这种语法而不是 arguments。

## 用法二：展开语法 (Spread syntax)

展开语法又可以在 3 个地方使用：

## 1）调用函数时

剩余参数是在函数定义时形参里使用的，展开语法的用处之一则是在函数**调用**时，传入的**实参**里使用。具体说就是将**数组表达式**或者 **string** 在语法层面展开作为实参传给函数。比如有如下代码：

```js
// 例 2.1
const arr = [1, 2, 3]
const string = 'Jay'

function fn(a, b, c) {
  console.log(a, b, c)
}
```

`fn` 接收 3 个参数，如果我们想把数组 `arr` 中的每个元素传给 `fn`，在 ES6 之前可以利用 apply 实现：

```js
// 例 2.1.1
fn.apply(null, arr) // 1 2 3
```

有了展开语法，则可以直接将作为参数的数组 `arr` 展开，这样可以提高代码的可读性 ：

```js
// 例 2.1.2
fn(...arr) // 1 2 3
```

展开语法还可以将字符串展开：

```js
// 例 2.1.3
fn(...string) // J a y
```

## 2）构造数组时

通过字面量方式构造数组时，也可以使用展开语法：

```js
// 例 2.2
const arr1 = [1, 2]
const arr2 = [3, 4, 5]
// 构造字面量数组
const newArr = [...arr1, ...arr2]
console.log(newArr) // [ 1, 2, 3, 4, 5 ]
```

## 3）构造字面量对象时

除了构造数组，ES2018(ES9) 添加了新的特性使得构造对象的时候也可以运用展开语法，被展开的对象表达式将按 key-value 的方式展开 ：

```js
// 例 2.3.1
const obj = { name: 'Jay' }
const newObj = { ...obj, age: 40 }
console.log(newObj) // { name: 'Jay', age: 40 }
```

在构造对象字面量时，还可以把数组也放进去展开，比如在 MDN 上关于展开语法的说明中有这么个例子：

```js
// 例 2.3.2
const obj1 = { foo: 'bar', x: 42 }
const obj2 = { foo: 'baz', y: 13 }
const merge = (...objects) => ({ ...objects })

const mergedObj = merge(obj1, obj2)
console.log(mergedObj) // { '0': { foo: 'bar', x: 42 }, '1': { foo: 'baz', y: 13 } }
```

乍看上去迷迷糊糊的，可能的难点在于第 4 行代码的理解，它定义了一个函数 `const merge = (...objects) => ({ ...objects }`， 其中 `=>` 左边的 `...objects` 为函数的剩余参数语法，所以 `objects` 就是一个由 obj1 和 obj2 这两个对象组成的数组：`[ { foo: 'bar', x: 42 }, { foo: 'baz', y: 13 } ]`；`=>` 右边的 `...objects` 用到的是展开语法，用于构造字面量对象，只不过因为 `objects` 是个数组，所以展开数组得到的对象的 key 为数组的下标，即 `{ ...objects }` 得到的对象为：`{ '0': { foo: 'bar', x: 42 }, '1': { foo: 'baz', y: 13 } }`。

### 注意

在调用函数或构造数组时使用展开语法只能用于**可迭代对象**，比如不能在构造数组时把一个对象放进去使用展开语法。

## 展开语法其实是一种浅拷贝

在 MDN 文档还能看到这么一句话：

> 实际上, 展开语法和 Object.assign() 行为一致, 执行的都是浅拷贝 (只遍历一层)。

也就是说类似下面的例 2.4：

```js
// 例 2.4
const obj = {
  a: 'a',
  b: {
    c: 'd'
  }
}
const newObj = { ...obj }
newObj.b.c = '我变了'
console.log(obj.b.c) // 我变了
```

我们改变通过展开语法得到的 newObj 的深层属性 `newObj.b.c = '我变了'` 时，会导致 `obj.b.c` 的值一同被修改。原因可以通过画内存表现图说明：

![](https://cdn.wallleap.cn/img/pic/illustration/202308041804525.jpeg)

`const newObj = { ...obj }` 生成的 newObj，只是把堆内存中 obj 指向的地址为 0x100 这个对象整个复制了一份，生成了地址为 0x300 的新对象（注意这个新对象里，b 的值是一个内存地址），然后让 newObj 指向它。如果我们更改了 newObj 的 a 属性或 b 属性的值，当然不会影响到 obj 对象，但是如果我们改变的是 `newObj.b.c`，就会找到 0x200 这个对象并修改，自然会影响到 obj 对象。

## 用法三：解构数组时

在对数组进行解构赋值时，我们可以使用剩余模式，将剩余数组赋值给一个变量，效果上类似于函数的剩余参数。比如下面例 3.1 中的 `c`：

```js
// 例 3.1
const arr = [1, 2, 3, 4, 5]
const [a, b, ...c] = arr
console.log(c) // [ 3, 4, 5 ]
```
