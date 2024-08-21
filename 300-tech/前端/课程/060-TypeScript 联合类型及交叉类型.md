---
title: 060-TypeScript 联合类型及交叉类型
date: 2023-09-11T01:50:00+08:00
updated: 2024-08-21T10:32:38+08:00
dg-publish: false
aliases:
  - TypeScript 联合类型及交叉类型
author: Luwang
category: 分类
comments: true
cover: //cdn.wallleap.cn/img/post/1.jpg
description: 文章描述
image-auto-upload: true
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
rating: 1
source: 
tags:
  - TypeScript
  - web
url: //myblog.wallleap.cn/post/1
---

TS = JS + type

JS 可以对值进行加减运算

如果把 TS 的**类型系统**当作一门语言，TS 可以对类型进行各种**运算**

- 联合类型 Union Types（并集）
- 交叉类型 Itersection Types（交集）

## 联合类型

![](https://cdn.wallleap.cn/img/pic/illustration/202309111355230.png)

### 如何使用联合类型

```ts
const fn = (a: number | string) => {
  既不能把 a 当作 number
  也不能把 a 当作 string
  那么应该怎样使用 a 变量呢
}
```

想办法把类型**区分开**

不拆开就只能使用 number 和 string 同时拥有的方法或属性，如 `toString()`

这个也叫**类型收窄**（Narrowing）

#### 使用 typeof 区分类型

```ts
const fn = (a: number | string) => {
  if(typeof a === 'number') {
    a.toFixed(2)
  } else if(typeof a === 'string') {
    parseFloat(a).toFixed(2)
  } else {
    throw new Error('Never do this')
  }
}
```

typeof x 的值可能是

`"string"` `"number"` `"bigint"` `"boolean"` `"symbol"` `undefined` `"object"` `"function"`

无法区分所有类型（局限性）

- 以下都会返回 `"object"`
	- typeof 数组对象
	- typeof 普通对象
	- typeof 日期对象
	- typeof null

#### 使用 instanceof 来区分类型

```ts
const fn = (a: Array<Date> | Date) => {
  if (a instanceof Date) {
    a.toISOString()
  } else if (a instanceof Array) {
    a[0].toISOString()
  } else {
    throw new Error('Never do this')
  }
}
```

instanceof 的局限性

- 不支持 string、number、boolean……（可以先用 typeof 先判断）
- 不支持 TS 独有的类型

#### 使用 in 来收窄类型

```ts
type Person = {
  name: string
}
const fn = (a: Person | Person[]) => {
  if('name' in a) {
    // Person
  } else {
    // Person []
  }
}
```

只适用于部分对象

#### 使用 JS 中判断类型的函数来区分

```ts
const fn = (a: string | string[]) => {
  if(Array.isArray(a)) {
    a.join('\n').toString() // string[]
  } else if (typeof a === 'string') {
    parseFloat(a).toFixed(2)
  } else {}
}
```

#### 使用逻辑来收窄类型

```ts
const f1 = (a?: string[]) => {
  if(a) {
    // string[]
  } else {
    // undefined
  }
}

const f2 = (a: string | number) => {
  a = 1
  // number
}
```

前面类型收窄都是通过 JS 实现，但是 JS 的类型系统很垃圾，有没有区分类型的完全之法

#### 类型谓词/类型判断 is

```ts
type Rect = { height: number; width: number }
type Circle = { center: [number, number]; radius: number }
function isRect(x: Rect | Circle): x is Rect {
  return 'height' in x && 'width' in x // return 中的可以使用上面那些
}
function isCircle(x: Rect | Circle): x is Circle {
  return 'center' in x && 'radius' in x
}
const fn = (a: Rect | Circle) => {
  if (isRect(a)) {
    // Rect
  } else {
    // Circle
  }
}
```

优点是支持所有 TS 类型，缺点是麻烦

有没有更简单的办法？

#### 可辨别联合 Discriminated Unions

假如给每个都加上同一标识 key

```ts
type A = {
  kind: 'string' // 也可以是其他
  value: string
}
type B = {
  kind: 'number'
  value: number
}

const fn = (a:A|B) => {
  if(a.kind === 'string') {
    // A
  } else {
    // B
  }
}
fn({ kinde: 'string', value: 'tom' })
```

但是 number 和 string 可以直接使用 typeof 区分，没必要

```ts
interface Circle { // 可以用 type
  kind: 'circle'
  radius: number
}
interface Square {
  kind: 'square'
  length: number
}
type Shape = Circle | Square

const fn = (shape: Shape) => {
  // 如果 shape: number | string | Shape
  // 可以先 typeof 判断再下面
  if (shape.kind === 'circle') {
    // Circle
  } else if (shape.kind === 'square') {
    // Square
  } else {
    // never
  }
}
```

优点：让**复杂类型**（对象）的收窄变成**简单类型**的对比

要求：

T = A | B | C | D | ...

1. A、B、C、D...**有相同属性** kind 或其他
2. kind 的类型是**简单类型**（字串、数字、布尔等）
3. 各类型中的 kind 要能够区分

满足上面三点可称 T 为可辨别联合

总结：同名、可辨别的简单类型的 key

使用场景

例如：

```ts
type Action = {type: 'getUser', id: string}
  | {type: 'createUser', attributes: any}
  | {type: 'deleteUser', id: string}
  | {type: 'updateUser', attributes: any}
```

#### 使用断言

知道类型，不需要判断

```ts
type Rect = { height: number; width: number }
type Circle = { center: [number, number]; radius: number }
type Shape = Rect | Circle

const fn = (a: Shape) => {
  (a as Circle).center
}
fn([1, 2], 3)
```

#### 疑问

1、any 是所有类型的联合吗（除了 never/unknown/any/void）？为什么？

不是，可以用反证法证明

any 可以使用所有类型的方法，但是类型联合使用的时候需要先进行收窄

不需要类型检查的时候使用 any，TS 绝大**部分规则**对 any 不生效

2、unknown = 所有类型的联合

先类型收窄

## 交叉类型

```ts
type A = string & number
// never
```

如上会发现交集不适合基础类型

![](https://cdn.wallleap.cn/img/pic/illustration/202309121409521.png)

交集需要所有的都写上

初始化的时候不能有多余的东西，但是可以把多余的赋值

```ts
const a:有左手的人 = {
  left: '一米八',
  right: '一米五' // 报错
}

const e = {
  left: '一米八',
  right: '一米五'
}
const f:有左手的人 = e
```

接口也能求交集

```ts
interface Colorful = {
  color: string
}
interface Circle {
  radius: number
}
type ColorfulCircle = Colorful & Circle
```

对象可以求交集，不在乎使用 type 或 interface

interface 可以直接 extend 继承，type 可以使用交集模拟继承

```ts
type Person = {
  name: string
  age: number
}
// 模拟 User 继承 Person
type User = Person & {
  id: number
  email: string
}
const u: User = {
  name: 'Tom',
  age: 18,
  id: 1,
  email: 'tom@qq.com'
}
```

& 左右是平等的，不存在顺序问题

交叉类型的特殊情况：如果有冲突

```ts
type Person = {
  id: string
  name: string
}
type User = {
  id: number
  email: string
} & Person

const a: User = {
  id: 1, // never
  name: 'Tom', // 其他可以用
  email: 'tom@qq.com'
}
```

但是如果用之前说的 kind 或其他唯一 key

```ts
type Person = {
  id: 'A'
  name: string
}
type User = {
  id: 'B'
  email: string
} & Person

const a: User = { // User 直接变成 never
  // 里面所有都不能写
  id: 1,
  name: 'Tom',
  email: 'tom@qq.com'
}
```

interface 和 type 区别 4：interface 继承会直接报错

```ts
interface Person {
  id: string
  name: string
}
interface User extends Person { // 报错，'number' 不能赋值到 'string'
  id: number
  // 收窄是可以的 id: '1'
  email: string
}
```

总结：对内使用 type，对外提供的接口使用 interface

发现里面是方法会很特殊

```ts
type A = {
  method: (n: number) => void
}
type B = {
  method: (n: string) => void
} & A

const b: B = {
  method: (n) => { // n 是 number | string
    console.log(n)
  }
}
```

函数的交集会得到参数的并集

结论：交叉类型常用于有交集的类型 A、B；如果 A、B 无交集，可能得到 never，也可能只是属性为 never

## 类型兼容与赋值

为什么要兼容：实际工作中，往往无法类型一致

例如：runTask 值接收 a、b、c

```ts
const config = {
  a: 1, b: 2, c: 3, d: 4
}

// 只要 a b c
const newConfig = lodash.pick(config, ['a', 'b', 'c'])
runTask(newConfig)

// 都传，多了也没关系
runTask(config)
```

类型兼容：你有的，我都有，则我可以**代替**你【y 有的，x 都有，则 x 兼容 y】

**简单类型**的兼容：小的（值）能赋值给大的

```ts
type A = string | number
let a: A = 'hi'
// 'hi' 是 string
// a 是 string | number
// 值 string 是小的，能兼容 string | number
// 能把小的，赋值给大的
```

**普通对象**的兼容：属性多的可以赋值给属性少的

```ts
type Person = {
  name: string
  age: number
}

const user = {
  name: 'Tom',
  age: 18,
  id: 1,
  email: 'tom@qq.com'
}
// type User = typeof user // 还可以这样反推类型
let p: Person
p = user
// user 兼容 p
// 故将 user 赋值给 p 不报错
// 属性多可以赋值给属性少的
```

父子接口兼容：子承父业，类似属性多的赋值给属性少的

```ts
interface 父接口 {
  x: string
}
interface 子接口 extends 父接口 { // 不用继承，有重叠的也可以
  y: string
}

let objectChild: 子接口 = {
  x: 'yes',
  y: 'yes'
}
let objectParent: 父接口
objectParent = objectChild
```

复杂的函数（参数和返回值）

参数个数不同，参数少的可以传给参数多的（容忍参数变少）

```ts
let 接收一个参数的函数 = (a: number) => {
  console.log(a)
}
let 接收两个参数的函数 = (b: number, s: string) => { // 赋值给上面会发现 s 没地方放，函数名也不同
  console.log(s, b)
}
接收两个参数的函数 = 接收一个参数的函数 // 可以的
接收一个参数的函数 = 接收两个参数的函数 // 报错
```

JS 中，少些参数是很常见的

```ts
const btn = document.getElementById('submit')
const fn = (e: MouseEvent) => console.log(e)
btn.addEventListener('click', fn)
btn.addEventListener('click', fn, true)

let items = [1, 2, 3]
items.forEach((item, index, array) => console.log(item))
items.forEach((item) => console.log(item))
```

参数的类型不同：对参数要求少的能传给对参数要求多的

```ts
interface MyEvent {
  target: string
}
interface MyMouseEvent extend MyEvent {
  x: number
  y: number
}

let listener = (e: MyEvent) => console.log(e.target)
let mouseListener = (e: MyMouseEvent) => console.log(e.x, e.y)

mouseListener = listener // Ok
listener = mouseListener // 报错
```

返回值类型

```ts
let 返回值属性少集合大 = () => {
  return { name: 'Alice' }
}
let 返回值属性多集合小 = () => {
  return { name: 'Alice', location: 'Seattle' }
}

返回值属性少集合大 = 返回值属性多集合小 // OK
返回值属性多集合小 = 返回值属性少集合大 // 'location' is missing
```

特殊类型

```ts
// any → any……
let a: any
let b: any = '1'
a = b

// any never
let a: never
let b: any = '1'
a = b // 报错
```

![赋值表](https://cdn.wallleap.cn/img/pic/illustration/202309131505312.png)

![](https://cdn.wallleap.cn/img/pic/illustration/202309131527609.png)
