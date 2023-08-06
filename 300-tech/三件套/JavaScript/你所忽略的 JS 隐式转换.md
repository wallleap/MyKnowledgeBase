---
title: 你所忽略的 JS 隐式转换
date: 2023-08-04 15:39
updated: 2023-08-04 15:44
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - 你所忽略的 JS 隐式转换
rating: 1
tags:
  - JS
  - web
category: web
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
description: 文章描述
source: #
url: //myblog.wallleap.cn/post/1
---

你有没有在面试中遇到特别奇葩的 js 隐形转换的面试题，第一反应是怎么会是这样呢？难以自信，js 到底是怎么去计算得到结果，你是否有深入去了解其原理呢？下面将深入讲解其实现原理。

其实这篇文章初稿三个月前就写好了，在我读一些源码库时，遇到了这些基础知识，想归档整理下，就有了这篇文章。由于一直忙没时间整理，最近看到了这个比较热的题，决定把这篇文章整理下。

```js
const a = {
  i: 1,
  toString: function () {
    return a.i++;
  }
}
if (a == 1 && a == 2 && a == 3) {
  console.log('hello world!');
}
```

网上给出了很多不错的解析过程，读了下面内容，你将更深入的了解其执行过程。

## 1、js 数据类型

js 中有 7 种数据类型，可以分为两类：原始类型、对象类型：

基础类型 (原始值)：

```js
// Undefined、 Null、 String、 Number、 Boolean、 Symbol (es6新出的，本文不讨论这种类型)
```

复杂类型 (对象值)：

```js
// object
```

## 2、三种隐式转换类型

js 中一个难点就是 js 隐形转换，因为 js 在一些操作符下其类型会做一些变化，所以 js 灵活，同时造成易出错，并且难以理解。

涉及隐式转换最多的两个运算符 \+ 和 \=\=。

- 运算符即可数字相加，也可以字符串相加。所以转换时很麻烦。\=\= 不同于\=\=\=，故也存在隐式转换。\- \\\* \/ 这些运算符只会针对 number 类型，故转换的结果只能是转换成 number 类型。

既然要隐式转换，那到底怎么转换呢，应该有一套转换规则，才能追踪最终转换成什么了。

隐式转换中主要涉及到三种转换：

1、将值转为原始值，ToPrimitive()。

2、将值转为数字，ToNumber()。

3、将值转为字符串，ToString()。

### 2.1、通过 ToPrimitive 将值转换为原始值

js 引擎内部的抽象操作 ToPrimitive 有着这样的签名：

ToPrimitive(input, PreferredType?)

input 是要转换的值，PreferredType 是可选参数，可以是 Number 或 String 类型。他只是一个转换标志，转化后的结果并不一定是这个参数所值的类型，但是转换结果一定是一个原始值（或者报错）。

#### 2.1.1、如果 PreferredType 被标记为 Number，则会进行下面的操作流程来转换输入的值

```
1、如果输入的值已经是一个原始值，则直接返回它
2、否则，如果输入的值是一个对象，则调用该对象的valueOf()方法，
   如果valueOf()方法的返回值是一个原始值，则返回这个原始值。
3、否则，调用这个对象的toString()方法，如果toString()方法返回的是一个原始值，则返回这个原始值。
4、否则，抛出TypeError异常。
```

#### 2.1.2、如果 PreferredType 被标记为 String，则会进行下面的操作流程来转换输入的值

```
1、如果输入的值已经是一个原始值，则直接返回它
2、否则，调用这个对象的toString()方法，如果toString()方法返回的是一个原始值，则返回这个原始值。
3、否则，如果输入的值是一个对象，则调用该对象的valueOf()方法，
   如果valueOf()方法的返回值是一个原始值，则返回这个原始值。
4、否则，抛出TypeError异常。
```

既然 PreferredType 是可选参数，那么如果没有这个参数时，怎么转换呢？PreferredType 的值会按照这样的规则来自动设置：

```
1、该对象为Date类型，则PreferredType被设置为String
2、否则，PreferredType被设置为Number
```

#### 2.1.3、valueOf 方法和 toString 方法解析

上面主要提及到了 valueOf 方法和 toString 方法，那这两个方法在对象里是否一定存在呢？答案是肯定的。在控制台输出 Object.prototype，你会发现其中就有 valueOf 和 toString 方法，而 Object.prototype 是所有对象原型链顶层原型，所有对象都会继承该原型的方法，故任何对象都会有 valueOf 和 toString 方法。

先看看对象的 valueOf 函数，其转换结果是什么？对于 js 的常见内置对象：`Date, Array, Math, Number, Boolean, String, Array, RegExp, Function`。

1、Number、Boolean、String 这三种构造函数生成的基础值的对象形式，通过 valueOf 转换后会变成相应的原始值。如：

```js
var num = new Number('123');
num.valueOf(); // 123

var str = new String('12df');
str.valueOf(); // '12df'

var bool = new Boolean('fd');
bool.valueOf(); // true
```

2、Date 这种特殊的对象，其原型 Date.prototype 上内置的 valueOf 函数将日期转换为日期的毫秒的形式的数值。

```js
var a = new Date();
a.valueOf(); // 1515143895500
```

3、除此之外返回的都为 this，即对象本身：(有问题欢迎告知)

```js
var a = new Array();
a.valueOf() === a; // true

var b = new Object({});
b.valueOf() === b; // true
```

再来看看 toString 函数，其转换结果是什么？对于 js 的常见内置对象：`Date, Array, Math, Number, Boolean, String, Array, RegExp, Function`。

1、Number、Boolean、String、Array、Date、RegExp、Function 这几种构造函数生成的对象，通过 toString 转换后会变成相应的字符串的形式，因为这些构造函数上封装了自己的 toString 方法。如：

```js
Number.prototype.hasOwnProperty('toString'); // true
Boolean.prototype.hasOwnProperty('toString'); // true
String.prototype.hasOwnProperty('toString'); // true
Array.prototype.hasOwnProperty('toString'); // true
Date.prototype.hasOwnProperty('toString'); // true
RegExp.prototype.hasOwnProperty('toString'); // true
Function.prototype.hasOwnProperty('toString'); // true

var num = new Number('123sd');
num.toString(); // 'NaN'

var str = new String('12df');
str.toString(); // '12df'

var bool = new Boolean('fd');
bool.toString(); // 'true'

var arr = new Array(1,2);
arr.toString(); // '1,2'

var d = new Date();
d.toString(); // "Wed Oct 11 2017 08:00:00 GMT+0800 (中国标准时间)"

var func = function () {}
func.toString(); // "function () {}"
```

除这些对象及其实例化对象之外，其他对象返回的都是该对象的类型，(有问题欢迎告知)，都是继承的 Object.prototype.toString 方法。

```js
var obj = new Object({});
obj.toString(); // "[object Object]"

Math.toString(); // "[object Math]"
```

从上面 valueOf 和 toString 两个函数对对象的转换可以看出为什么对于 ToPrimitive(input, PreferredType?)，PreferredType 没有设定的时候，除了 Date 类型，PreferredType 被设置为 String，其它的会设置成 Number。

因为 valueOf 函数会将 Number、String、Boolean 基础类型的对象类型值转换成 基础类型，Date 类型转换为毫秒数，其它的返回对象本身，而 toString 方法会将所有对象转换为字符串。显然对于大部分对象转换，valueOf 转换更合理些，因为并没有规定转换类型，应该尽可能保持原有值，而不应该想 toString 方法一样，一股脑将其转换为字符串。

所以对于没有指定 PreferredType 类型时，先进行 valueOf 方法转换更好，故将 PreferredType 设置为 Number 类型。

而对于 Date 类型，其进行 valueOf 转换为毫秒数的 number 类型。在进行隐式转换时，没有指定将其转换为 number 类型时，将其转换为那么大的 number 类型的值显然没有多大意义。（不管是在 \+ 运算符还是 \=\= 运算符）还不如转换为字符串格式的日期，所以默认 Date 类型会优先进行 toString 转换。故有以上的规则：

PreferredType 没有设置时，Date 类型的对象，PreferredType 默认设置为 String，其他类型对象 PreferredType 默认设置为 Number。

### 2.2、通过 ToNumber 将值转换为数字

根据参数类型进行下面转换：

| 参数 | 结果 |
| --- | --- |
| undefined | NaN |
| null | +0 |
| 布尔值 | true 转换 1，false 转换为 +0 |
| 数字 | 无须转换 |
| 字符串 | 有字符串解析为数字，例如：‘324’转换为 324，‘qwer’转换为 NaN |
| 对象 (obj) | 先进行 ToPrimitive(obj, Number) 转换得到原始值，在进行 ToNumber 转换为数字 |

### 2.3、通过 ToString 将值转换为字符串

根据参数类型进行下面转换：

| 参数 | 结果 |
| --- | --- |
| undefined | 'undefined' |
| null | 'null' |
| 布尔值 | 转换为 'true' 或 'false' |
| 数字 | 数字转换字符串，比如：1.765 转为 '1.765' |
| 字符串 | 无须转换 |
| 对象 (obj) | 先进行 ToPrimitive(obj, String) 转换得到原始值，在进行 ToString 转换为字符串 |

讲了这么多，是不是还不是很清晰，先来看看一个例子：

```
({} + {}) = ?
两个对象的值进行+运算符，肯定要先进行隐式转换为原始类型才能进行计算。
1、进行ToPrimitive转换，由于没有指定PreferredType类型，{}会使默认值为Number，进行ToPrimitive(input, Number)运算。
2、所以会执行valueOf方法，({}).valueOf(),返回的还是{}对象，不是原始值。
3、继续执行toString方法，({}).toString(),返回"[object Object]"，是原始值。
故得到最终的结果，"[object Object]" + "[object Object]" = "[object Object][object Object]"
```

再来一个指定类型的例子：

```
2 * {} = ?
1、首先*运算符只能对number类型进行运算，故第一步就是对{}进行ToNumber类型转换。
2、由于{}是对象类型，故先进行原始类型转换，ToPrimitive(input, Number)运算。
3、所以会执行valueOf方法，({}).valueOf(),返回的还是{}对象，不是原始值。
4、继续执行toString方法，({}).toString(),返回"[object Object]"，是原始值。
5、转换为原始值后再进行ToNumber运算，"[object Object]"就转换为NaN。
故最终的结果为 2 * NaN = NaN
```

## 3、\=\= 运算符隐式转换

**\=\= 运算符的规则规律性不是那么强，按照下面流程来执行，es5 文档**

```
比较运算 x==y, 其中 x 和 y 是值，返回 true 或者 false。这样的比较按如下方式进行：

1、若 Type(x) 与 Type(y) 相同， 则

    1* 若 Type(x) 为 Undefined， 返回 true。
    2* 若 Type(x) 为 Null， 返回 true。
    3* 若 Type(x) 为 Number， 则
  
        (1)、若 x 为 NaN， 返回 false。
        (2)、若 y 为 NaN， 返回 false。
        (3)、若 x 与 y 为相等数值， 返回 true。
        (4)、若 x 为 +0 且 y 为 −0， 返回 true。
        (5)、若 x 为 −0 且 y 为 +0， 返回 true。
        (6)、返回 false。
        
    4* 若 Type(x) 为 String, 则当 x 和 y 为完全相同的字符序列（长度相等且相同字符在相同位置）时返回 true。 否则， 返回 false。
    5* 若 Type(x) 为 Boolean, 当 x 和 y 为同为 true 或者同为 false 时返回 true。 否则， 返回 false。
    6*  当 x 和 y 为引用同一对象时返回 true。否则，返回 false。
  
2、若 x 为 null 且 y 为 undefined， 返回 true。
3、若 x 为 undefined 且 y 为 null， 返回 true。
4、若 Type(x) 为 Number 且 Type(y) 为 String，返回比较 x == ToNumber(y) 的结果。
5、若 Type(x) 为 String 且 Type(y) 为 Number，返回比较 ToNumber(x) == y 的结果。
6、若 Type(x) 为 Boolean， 返回比较 ToNumber(x) == y 的结果。
7、若 Type(y) 为 Boolean， 返回比较 x == ToNumber(y) 的结果。
8、若 Type(x) 为 String 或 Number，且 Type(y) 为 Object，返回比较 x == ToPrimitive(y) 的结果。
9、若 Type(x) 为 Object 且 Type(y) 为 String 或 Number， 返回比较 ToPrimitive(x) == y 的结果。
10、返回 false。
```

上面主要分为两类，x、y 类型相同时，和类型不相同时。

类型相同时，没有类型转换，主要注意 NaN 不与任何值相等，包括它自己，即 NaN !== NaN。

类型不相同时，

1、x,y 为 null、undefined 两者中一个 // 返回 true

2、x、y 为 Number 和 String 类型时，则转换为 Number 类型比较。

3、有 Boolean 类型时，Boolean 转化为 Number 类型比较。

4、一个 Object 类型，一个 String 或 Number 类型，将 Object 类型进行原始转换后，按上面流程进行原始值比较。

### 3.1、\=\= 例子解析

所以类型不相同时，可以会进行上面几条的比较，比如：

```js
var a = {
  valueOf: function () {
     return 1;
  },
  toString: function () {
     return '123'
  }
}
true == a // true;
首先，x与y类型不同，x为boolean类型，则进行ToNumber转换为1,为number类型。
接着，x为number，y为object类型，对y进行原始转换，ToPrimitive(a, ?),没有指定转换类型，默认number类型。
而后，ToPrimitive(a, Number)首先调用valueOf方法，返回1，得到原始类型1。
最后 1 == 1， 返回true。
```

我们再看一段很复杂的比较，如下：

```
[] == !{}
//
1、! 运算符优先级高于==，故先进行！运算。
2、!{}运算结果为false，结果变成 [] == false比较。
3、根据上面第7条，等式右边y = ToNumber(false) = 0。结果变成 [] == 0。
4、按照上面第9条，比较变成ToPrimitive([]) == 0。
    按照上面规则进行原始值转换，[]会先调用valueOf函数，返回this。
   不是原始值，继续调用toString方法，x = [].toString() = ''。
   故结果为 '' == 0比较。
5、根据上面第5条，等式左边x = ToNumber('') = 0。
   所以结果变为： 0 == 0，返回true，比较结束。
```

最后我们看看文章开头说的那道题目：

```js
const a = {
  i: 1,
  toString: function () {
    return a.i++;
  }
}
if (a == 1 && a == 2 && a == 3) {
  console.log('hello world!');
}
```

1、当执行 `a == 1 && a == 2 && a == 3` 时，会从左到右一步一步解析，首先 `a == 1`，会进行上面第 9 步转换。`ToPrimitive(a， Number) == 1`。

2、`ToPrimitive(a, Number)`，按照上面原始类型转换规则，会先调用 valueOf 方法，a 的 valueOf 方法继承自 Object.prototype。返回 a 本身，而非原始类型，故会调用 toString 方法。

3、因为 toString 被重写，所以会调用重写的 toString 方法，故返回 1，注意这里是 i++，而不是 ++i，它会先返回 i，在将 i+1。故 ToPrimitive(a, Number) = 1。也就是 `1 == 1`，此时 `i = 1 + 1 = 2`。

4、执行完 `a == 1` 返回 true，会执行 `a == 2`，同理，会调用 ToPrimitive(a, Number)，同上先调用 valueOf 方法，在调用 toString 方法，由于第一步，i = 2 此时，ToPrimitive(a, Number) = 2， 也就是 `2 == 2`, 此时 i = 2 + 1。

5、同上可以推导 `a == 3` 也返回 true。故最终结果 `a == 1 && a == 2 && a == 3` 返回 true

其实了解了以上隐形转换的原理，你有没有发现这些隐式转换并没有想象中那么难。

## 参考文章：[es5文档](https://link.juejin.cn/?target=http%3A%2F%2Fyanhaijing.com%2Fes5%2F%23103 "http://yanhaijing.com/es5/#103")
