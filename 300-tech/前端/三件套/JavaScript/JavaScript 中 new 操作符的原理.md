---
title: JavaScript 中 new 操作符的原理
date: 2023-08-04T05:32:00+08:00
updated: 2024-08-21T10:33:28+08:00
dg-publish: false
aliases:
  - JavaScript 中 new 操作符的原理
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

## 🎯总结

1. 创建一个**空对象**
2. 空对象的内部属性 `__proto__` 赋值为构造函数的 `prototype` 属性
3. 将构造函数的 `this` 指向空对象
4. 执行构造函数内部代码
5. 返回该新对象

## 详细说明

执行 new 操作时会依次经过以下步骤：

1、创建一个**空对象**

空对象是 Object 的实例，即 `{}` 。

```js
let obj = {}
```

2、空对象的内部属性 `__proto__` 赋值为构造函数的 `prototype` 属性

该操作是为了将空对象链接到正确的原型上去

```js
function Foo(num) {
  this.number = num
}
obj.__proto__ = Foo.prototype
```

3、将构造函数的 `this` 指向空对象

即构造函数内部的 this 被赋值为空对象，以便后面正确执行构造函数。

```js
Foo.call(obj, 1)
```

4、执行构造函数内部代码

即给空对象添加属性、方法。

5、返回该新对象

- 如果构造函数内部通过 return 语句返回了一个引用类型值，则 new 操作最终返回这个引用类型值；否则返回刚创建的新对象。
- 引用类型值：除基本类型值（数值、字符串、布尔值、null、undefined、Symbol 值）以外的所有值。

## 模拟 new 操作符

下面的 `myNew` 函数模拟了 new 操作符的行为

```js
function myNew(func, ...args) {
  let obj = {}
  obj.__proto__ = func.prototype
  let res = func.apply(obj, args)
  return res instanceof Object ? res : obj
}

function Foo(num) {
  this.number = num
}

let foo1 = myNew(Foo, 1)
console.log(foo1 instanceof Foo)  // true
console.log(foo1.number)  // 1

let foo2 = new Foo(2)
console.log(foo2 instanceof Foo)  // true
console.log(foo2.number)  // 2
```

上面通过 `instanceof` 操作符来判断构造函数的返回值是否为 `Object` 的实例，即是否为引用类型值；这是因为所有引用类型值都是 Object 的实例，Object 是所有引用类型值的基类型。
