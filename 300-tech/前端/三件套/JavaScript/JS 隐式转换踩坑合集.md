---
title: JS 隐式转换踩坑合集
date: 2023-08-04T03:34:00+08:00
updated: 2024-08-21T10:33:28+08:00
dg-publish: false
aliases:
  - JS 隐式转换踩坑合集
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

> 前方提醒： 篇幅较长，点个赞或者收藏一下，可以在下一次阅读时方便查找

提到 js 的隐式转换，很多人第一反应都是：坑。

的确，对于不熟悉的人来说，`js隐式转换` 存在着很多的让人无法预测的地方，相信很多人都深受其害，所以，大家在开发过程中，可能会使用 \=\=\= 来尽量避免隐式转换。但是，为了更加深入的理解 `javascript`，本着对知识渴望的精神，我们来通过大量的例子分析分析 `js隐式转换`，熟悉 `js隐式转换` 的规则，让其在你的眼里变成“显式”。

## 从一道面试题说起

先来看看一个经典的面试题

> 定义一个变量 `a`，使得下面的表达式结果为 `true`

```
a == 1 && a == 2 && a == 3
```

还有这种操作？先试试看吧，定义 `a = true`?

```js
var a = true
a == 1 && a == 2 && a == 3 // false
```

> 但是并没有达到预期，好像触碰到知识盲区了。。。没关系，先放下吧，来看看几个更坑的

```js
[] == ![] // true

[] == 0 // true

[2] == 2 // true

['0'] == false // true

'0' == false // true

[] == false // true

[null] == 0 // true

null == 0 // false

[null] == false // true

null == false // false

[undefined] == false // true

undefined == false // false
```

一脸懵逼？ 不要紧！接下来带你完完全全的认识 `javascript的隐式转换`。

## javascript 隐式转换规则

### 1\. ToString，ToNumber，ToBoolean，ToPrimitive

我们需要先了解一下 js 数据类型之间转换的基本规则，比如数字、字符串、布尔型、数组、对象之间的相互转换。

#### 1.1 ToString

> 这里所说的 `ToString` 可不是对象的 `toString方法`，而是指其他类型的值转换为字符串类型的操作。

这里我们讨论 `null`、`undefined`、`布尔型`、`数字`、`数组`、`普通对象` 转换为字符串的规则。

- null：转为 `"null"`
- undefined：转为 `"undefined"`
- 布尔类型：`true` 和 `false` 分别被转为 `"true"` 和 `"false"`
- 数字类型：转为数字的字符串形式，如 `10` 转为 `"10"`， `1e21` 转为 `"1e+21"`
- 数组：转为字符串是将所有元素按照 "," 连接起来，相当于调用数组的 `Array.prototype.join()` 方法，如 `[1, 2, 3]` 转为 `"1,2,3"`，空数组 `[]` 转为空字符串，数组中的 `null` 或 `undefined`，会被当做空字符串处理
- 普通对象：转为字符串相当于直接使用 `Object.prototype.toString()`，返回 `"[object Object]"`

```js
String(null) // 'null'
String(undefined) // 'undefined'
String(true) // 'true'
String(10) // '10'
String(1e21) // '1e+21'
String([1,2,3]) // '1,2,3'
String([]) // ''
String([null]) // ''
String([1, undefined, 3]) // '1,,3'
String({}) // '[object Objecr]'
```

对象的 `toString` 方法，满足 `ToString` 操作的规则。

> 注意：上面所说的规则是在默认的情况下，如果修改默认的 `toString()` 方法，会导致不同的结果

#### 1.2 ToNumber

> `ToNumber` 指其他类型转换为数字类型的操作。

- null： 转为 `0`
- undefined：转为 `NaN`
- 字符串：如果是纯数字形式，则转为对应的数字，空字符转为 `0`, 否则一律按转换失败处理，转为 `NaN`
- 布尔型：`true` 和 `false` 被转为 `1` 和 `0`
- 数组：数组首先会被转为原始类型，也就是 `ToPrimitive`，然后在根据转换后的原始类型按照上面的规则处理，关于 `ToPrimitive`，会在下文中讲到
- 对象：同数组的处理

```js
Number(null) // 0
Number(undefined) // NaN
Number('10') // 10
Number('10a') // NaN
Number('') // 0 
Number(true) // 1
Number(false) // 0
Number([]) // 0
Number(['1']) // 1
Number({}) // NaN
```

#### 1.3 ToBoolean

> `ToBoolean` 指其他类型转换为布尔类型的操作。

js 中的假值只有 `false`、`null`、`undefined`、`空字符`、`0` 和 `NaN`，其它值转为布尔型都为 `true`。

```js
Boolean(null) // false
Boolean(undefined) // false
Boolean('') // flase
Boolean(NaN) // flase
Boolean(0) // flase
Boolean([]) // true
Boolean({}) // true
Boolean(Infinity) // true
```

#### 1.4 ToPrimitive

> `ToPrimitive` 指对象类型类型（如：对象、数组）转换为原始类型的操作。

- 当对象类型需要被转为原始类型时，它会先查找对象的 `valueOf` 方法，如果 `valueOf` 方法返回原始类型的值，则 `ToPrimitive` 的结果就是这个值
- 如果 `valueOf` 不存在或者 `valueOf` 方法返回的不是原始类型的值，就会尝试调用对象的 `toString` 方法，也就是会遵循对象的 `ToString` 规则，然后使用 `toString` 的返回值作为 `ToPrimitive` 的结果。

> 注意：对于不同类型的对象来说，`ToPrimitive` 的规则有所不同，比如 `Date对象` 会先调用 `toString`，具体可以参考 [ECMA标准](https://link.juejin.cn/?target=https%3A%2F%2Fwww.ecma-international.org%2Fecma-262%2F6.0%2F%23sec-toprimitive "https://www.ecma-international.org/ecma-262/6.0/#sec-toprimitive")

如果 `valueOf` 和 `toString` 都没有返回原始类型的值，则会抛出异常。

```js
Number([]) // 0
Number(['10']) //10

const obj1 = {
  valueOf () {
    return 100
  },
  toString () {
    return 101
  }
}
Number(obj1) // 100

const obj2 = {
  toString () {
    return 102
  }
}
Number(obj2) // 102

const obj3 = {
  toString () {
    return {}
  }
}
Number(obj3) // TypeError
```

前面说过，对象类型在 `ToNumber` 时会先 `ToPrimitive`，再根据转换后的原始类型 `ToNumber`

- `Number([])`， 空数组会先调用 `valueOf`，但返回的是数组本身，不是原始类型，所以会继续调用 `toString`，得到 `空字符串`，相当于 `Number('')`，所以转换后的结果为 `"0"`
- 同理，`Number(['10'])` 相当于 `Number('10')`，得到结果 `10`
- `obj1` 的 `valueOf` 方法返回原始类型 `100`，所以 `ToPrimitive` 的结果为 `100`
- `obj2` 没有 `valueOf`，但存在 `toString`，并且返回一个原始类型，所以 `Number(obj2)` 结果为 `102`
- `obj3` 的 `toString` 方法返回的不是一个原始类型，无法 `ToPrimitive`，所以会抛出错误

> 看到这里，以为自己完全掌握了？别忘了，那道面试题和那一堆让人懵逼的判断还没解决呢，本着对知识渴望的精神，继续往下看吧。

### 2\. 宽松相等（\=\=）比较时的隐式转换规则

`宽松相等（==）` 和 `严格相等（===）` 的区别在于宽松相等会在比较中进行 `隐式转换`。现在我们来看看不同情况下的转换规则。

#### 2.1 布尔类型和其他类型的相等比较

- 只要 `布尔类型` 参与比较，该 `布尔类型` 的值首先会被转换为 `数字类型`
- 根据 `布尔类型` 的 `ToNumber` 规则，`true` 转为 `1`，`false` 转为 `0`

```
false == 0 // true
true == 1 // true
true == 2 // false
```

> 之前有的人可能觉得数字 `2` 是一个真值，所以 `true == 2` 应该为真，现在明白了，布尔类型 `true` 参与相等比较会先转为数字 `1`，相当于 `1 == 2`，结果当然是 `false`

我们平时在使用 `if` 判断时，一般都是这样写

```js
const x = 10
if (x) {
  console.log(x)
}
```

这里 `if(x)` 的 `x` 会在这里被转换为布尔类型，所以代码可以正常执行。但是如果写成这样：

```js
const x = 10
if (x == true) {
  console.log(x)
}
```

代码不会按照预期执行，因为 `x == true` 相当于 `10 == 1`

#### 2.2 数字类型和字符串类型的相等比较

- 当 `数字类型` 和 `字符串类型` 做相等比较时，`字符串类型` 会被转换为 `数字类型`
- 根据字符串的 `ToNumber` 规则，如果是纯数字形式的字符串，则转为对应的数字，空字符转为 `0`, 否则一律按转换失败处理，转为 `NaN`

```
0 == '' // true
1 == '1' // true
1e21 == '1e21' // true
Infinity == 'Infinity' // true
true == '1' // true
false == '0' // true
false == '' // true
```

上面比较的结果和你预期的一致吗？ 根据规则，字符串转为数字，布尔型也转为数字，所以结果就显而易见了。

> 这里就不讨论 `NaN` 了，因为 `NaN` 和任何值都不相等，包括它自己。

#### 2.3 对象类型和原始类型的相等比较

- 当 `对象类型` 和 `原始类型` 做相等比较时，`对象类型` 会依照 `ToPrimitive` 规则转换为 `原始类型`

```
'[object Object]' == {} // true
'1,2,3' == [1, 2, 3] // true
```

看一下文章开始时给出的例子

```
[2] == 2 // true
```

数组 `[2]` 是对象类型，所以会进行 `ToPrimitive` 操作，也就是先调用 `valueOf` 再调用 `toString`，根据数组 `ToString` 操作规则，会得到结果 `"2"`， 而字符串 `"2"` 再和数字 `2` 比较时，会先转为数字类型，所以最后得到的结果为 `true`。

```
[null] == 0 // true
[undefined] == 0 // true
[] == 0 // true
```

根据上文中提到的数组 `ToString` 操作规则，数组元素为 `null` 或 `undefined` 时，该元素被当做 `空字符串` 处理，而空数组 `[]` 也被转为 `空字符串`，所以上述代码相当于

```
'' == 0 // true
'' == 0 // true
'' == 0 // true
```

`空字符串` 会转换为数字 `0`，所以结果为 `true`。

试试 valueOf 方法

```js
const a = {
  valueOf () {
    return 10
  }
  toString () {
    return 20
  }
}
a == 10 // true
```

> 对象的 `ToPrimitive` 操作会先调用 `valueOf` 方法，并且 `a` 的 `valueOf` 方法返回一个原始类型的值，所以 `ToPrimitive` 的操作结果就是 `valueOf` 方法的返回值 `10`。

讲到这里，你是不是想到了最开始的面试题？ 对象每次和原始类型做 \=\= 比较时，都会进行一次 `ToPrimitive` 操作，那我们是不是可以定义一个包含 `valueOf` 方法的对象，然后通过某个值的累加来实现？

试一试

```js
const a = {
  // 定义一个属性来做累加
  i: 1,
  valueOf () {
    return this.i++
  }
}
a == 1 && a == 2 && a == 3 // true
```

结果正如你所想的，是正确的。当然，当没有定义 `valueOf` 方法时，用 `toString` 方法也是可以的。

```js
const a = {
  // 定义一个属性来做累加
  i: 1,
  toString () {
    return this.i++
  }
}
a == 1 && a == 2 && a == 3 // true
```

#### 2.4 null、undefined 和其他类型的比较

- `null` 和 `undefined` 宽松相等的结果为 true，这一点大家都知道

其次，`null` 和 `undefined` 都是假值，那么

```js
null == false // false
undefined == false // false
```

居然跟我想的不一样？为什么呢？ 首先，`false` 转为 `0`，然后呢？ 没有然后了，`ECMAScript规范` 中规定 `null` 和 `undefined` 之间互相 `宽松相等（==）`，并且也与其自身相等，但和其他所有的值都不 `宽松相等（==）`。

## 最后

现在再看前面的这一段代码就明了了许多

```js
[] == ![] // true

[] == 0 // true

[2] == 2 // true

['0'] == false // true

'0' == false // true

[] == false // true

[null] == 0 // true

null == 0 // false

[null] == false // true

null == false // false

[undefined] == false // true

undefined == false // false
```

最后想告诉大家，不要一味的排斥 javascript 的隐式转换，应该学会如何去利用它，你的代码中可能存在着很多的隐式转换，只是你忽略了它，要做到知其然，并知其所以然，这样才能有助于我们深入的理解 javascript。

（看了这么久了，辛苦了，不过我也写了很久啊，点个赞再走吧）
