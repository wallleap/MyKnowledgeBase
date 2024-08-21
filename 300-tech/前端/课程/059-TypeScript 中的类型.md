---
title: 059-TypeScript 中的类型
date: 2023-09-08T02:03:00+08:00
updated: 2024-08-21T10:32:38+08:00
dg-publish: false
aliases:
  - TypeScript 中的类型
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

## JS/TS 中的数据类型

### JS datatype

null undefined

string number boolean

bigint symbol

object（含 Array、Function、Date...）

### TS datatype

- 以上所有
- 加上 void never enum unknown any
- 再加上自定义类型 type interface

## 如何理解 TS 的数据类型

### 从**集合**的角度理解

![](https://cdn.wallleap.cn/img/pic/illustration/202309081410390.png)

![](https://cdn.wallleap.cn/img/pic/illustration/202309081411970.png)

### Object 中的 String Number Boolean 不使用

> 基本类型中有 number string boolean，对象中有 String Number Boolean

`let n:numer = 42` V.S. `let n = new Number(42)`

左边是以二进制存储

右边是除了有数字还有方法

![](https://cdn.wallleap.cn/img/pic/illustration/202309081414486.png)

JS 包装对象，会将前面的变成后面的

![](https://cdn.wallleap.cn/img/pic/illustration/202309081416173.png)

总结：JS 中的 Number String Boolean 只用于包装对象，正常开发者用不到他们，在 TS 里也不用（只使用小写的）

### type Object 表示的范围太大，不使用

这些值都是 Object

![](https://cdn.wallleap.cn/img/pic/illustration/202309081421668.png)

- 对象
	- 键为字串/数字、值为数字/字符串
	- 还可以是空对象
- 数组
	- 长度不同
- 函数
	- 有参数
	- 有返回值
- Date
- 正则

> 故在 TS 中一般不用 Object

如何在 TS 里**描述对象**的数据类型（datatype）

1. 使用 class（类）/constructor（构造函数）描述，例如 Date、Array、Function 等
	![](https://cdn.wallleap.cn/img/pic/illustration/202309081428752.png)
2. 用 type 或 interface 描述（推荐，会更具体）

例如声明一个类型

```ts
type Person = {
  name: string
  age: number
}
```

这样更具体，只能有这两个 key，类型分别为 string 和 number

两个之间可以使用 `,`、`;`、回车连接（格式要求很松散）

```ts
const a:Person = {
  name: 'Tom', // 这里还是对象的语法，使用逗号
  age: 18
} 
```

对象字面量就只能有 name 和 age，并且不能多也不能少，类型需要匹配

如果不需要限制字段，可以这样（索引签名）

```ts
type A = {
  [k: string]: number
}

const a: A = {
  name: 1,
  age: 6,
  233: 22 // JS 中前面 233 最终还是 string
}
```

![](https://cdn.wallleap.cn/img/pic/illustration/202309081436011.png)

key 的类型可以不是 string，不是的时候就需要匹配（字面上的检查）

string、number、symbol

```ts
type A = {
  [k: number]: string
}
const a: A = {
  123: '111' // name: '111' 字面上是 string 会报错
}

type B = {
  [key: symbo]: number
}
const s = Symbol()
const b: B = {
  [s]: 1 // JS 中如果只填 s 就是 string，这里明显是变量，需要用 [] 括起来
}
```

还可以写成这样

```ts
type A2 = Record<string, number>
```

![](https://cdn.wallleap.cn/img/pic/illustration/202309081438809.png)

由于 Object 太不精确（包装类也能赋值），所以 TS 开发者一般使用**索引签名**或 **Record 泛型**来描述**普通对象**

**数组**对象，如何描述？

![](https://cdn.wallleap.cn/img/pic/illustration/202309081515686.png)

```ts
type A = string[]
type B = Array<number>
type C = [string, number]
type D = [1,2,3]

const a:A = ['2']
const b:B = [1,2,34,4,5]
const c:C = ['1', 2]
const d:D = [1,2,3]
```

元组：指定具体多少个，每个的类型也指定

由于 Array 太不精确，所以 TS 开发者一般用 **`Array<?>`** 或 **`string[]`** 或 **`[string, number]`** 来描述数组

**函数**对象，如何描述？

![](https://cdn.wallleap.cn/img/pic/illustration/202309081528387.png)

```ts
type FnA = (a:number, b:number) => number
const a:FnA = () => {
  return 123
}
const b:FnA = (x) => {
  return 123
}
const c:FnA = (x, y) => {
  return 123
}
const d:FnA = (x:number, y:number) => {
  return 123
}
a()
b(1)
c(1,2)
d(1,2)
```

上面都可以，如果错了，注意看报错

**void 没有返回值**（JS 自动加 return undefined，但是 TS 不是），如果返回值是 undefined 需要显式地 return undefined

声明支持 this 的函数类型：

```ts
type Person = { username:string; age:number; sayHi:FnWithThis } // sayHi 放到对象中只是为了能 x.sayHi() 进行调用
type FnWithThis = (this:Person, username:string) => void

const sayHi:FnWithThis = function(username:string){ // 一般不用箭头函数（没有自己的 this）
  console.log('Hi ' + this.username + '! I\'m ' + username)
}
const x: Person = {
  username: 'Tom',
  age: 18,
  sayHi: sayHi
}
x.sayHi('Jack') // this 是 x
sayHi.call(x, 'Jack')
```

由于 Function 太不精确，所以使用 `()=> ?` 来描述函数

其他对象一般直接使用 **class**（构造函数）描述

```ts
const d:Date = new Date() // 不写可以通过右边进行推断
const r:RegExp = /ab+c/
const r2:RegExp = new RegExp('ab+c')
const m: Map<string, number> = new Map() // 有参数就不能删
m.set('xxx', 2)
const wm:WeakMap<{name:string}, number> = new WeakMap()
const s:Set<number> = new set()
s.add(123)
const ws:WeakSet<string[]> = new WeakSet()
```

小写的 object 就是除了基础类型之外的类型（包装类）

DOM 对象一般是已经定义了的（lib.dom.d.ts 中）

### any 和 unknown

将所有的类型并起来（number | string | boolean ...）就是 any

不知道是什么就是 unknown（我也不知道，后面自己断言它是什么类型）

```ts
const a:unkown = 1 // a.toFixed(2) 是不行的，类型从右边推到了
const b = (a as number).toFixed(2)

const res:unknown = await fetch('/api/users')
const data = (a as number)
```

一般不用 any，除非自己实在写不出这个类型是什么

### never

any 是所有的

never 是没有（空集），可以用作检查

never 不是声明的是推导出的

```ts
type A = string & number // 这就是空集

type B = string | number
const b:B = ('hello w' as any)
if(typeof b === 'string') {
  b.split('')
} else if(typeof b === 'number') {
  b.toFixed(2)
} else {
  // b.xxx() // 这里 b 就会是 never
  console.log('没有')
}
```

### enum 枚举

何时用 enum？

1、后端传了 status，值为 1-4，分别代表 todo、done、archived、deleted

前端获取到 status 为数字 1-4，前端显示分别对应未完成、已完成、归档、不做了，并且可以进行修改

例如：

```ts
enum A {
  todo = 1, // 必须写等于多少，后面的将自动递增
  done, // 逗号分隔
  archived,
  deleted
} // 本质还是数字，可以使用 A.xxx 将数字映射成有意义的语法

let status:A = 1
status = A.todo // 会被替换成 1
status = A.done
```

2、权限控制的时候

```ts
enum Permission { // 使用二进制位表示权限
  None = 0,         // 0000
  Read = 1 << 0, // 二进制左移 0 位 0001
  Write = 1 << 1,  // 0010
  Delete = 1 << 2, // 0100
  Mange = Read | Write | Delete // 0111
}

type User = {
  permission: Permission;
}
const user: User = {
  permission: 0b0101
}
// 若 a & b === b，则 a 有 b 的所有 1
if((user.permission & Permission.Write) === Permission.Write) {
  console.log('拥有写权限')
}
if((user.permission & Permission.Manage) === Permission.Manage) {
  console.log('拥有管理权限')
}
```

何时不用 enum？

用来表示字符串

```ts
enum Fruit {
  apple = 'apple',
  banana = 'banana',
  pineapple = 'pineaaple',
  watermelon = 'watermelon'
}

// let f: Fruit = 'apple' // 字符串就不能这样赋值，上面数字都可以
let f: Fruit = Fruit.apple
f = Fruit.watemelon
```

可以使用 type

```ts
type Fruit = 'apple' | 'banana' | 'pineaaple' | 'watermelon' // 并
}

let f: Fruit = 'apple'
f = 'watemelon'
```

总结：数字的时候适合使用 enum，其他就不太适合了

之前 JS 实现使用对象

```js
var map = {
  '00': 'read',
  '01': 'write'
}
```

### type interface

#### 类型别名 type alias

给其他类型取个名字

```ts
type Name = string
const a: Name = 'Tom'

type FalseLike = '' | 0 | null | undefined | false // | NaN 这个不是类型 假值

type Point = { x: number; y: number }
type Points = Point[]
type Line = [Point, Point]
type Circle = { center: Point; radius: number }

type Fn = (a: number, b: number) => number
const f: Fn = (x, y) {
  return x + y
}

type FnWithProps = {
  (a: number, b: number): number // 函数 (): void
  prop1: number
} // 有属性的函数
const f1: FnWithProp = (x, y) => {
  return x * y
}
f1.prop1 = 'hello'
```

React 中很常见

```ts
const FC: React.FC = () => {}
FC.defaultProps = {}
```

几乎万能的，可以使用 type 声明几乎所有的类型

叫类型**别名**的原因是类型名本来没想被记住的，只是给类型取了个其他名而已

#### 声明接口

描述对象的属性（declare the shapes of objects）

面向对象（Oriented Object）

- 编程 Program
	- class、方法、继承
	- 对应 JS 中 构造函数、函数、原型

interface 既能描述功能又能描述属性

```ts
interface X {
  a: number,
  b: string
}
interface Data {
  [k:string]: number // 索引签名
}
interface A extends Array<string>, X { // 继承 并拥有自己的属性
  name: string
}
type A1 = Array<string> & {
  name: string
} & X

interface Fn {
  (a:number, b:number): number
  xxx: number
}
const f:Fn = (x,y) => { return x + y }
f.xxx = 100

interface D extends Date {
}
const d: D = new Date()

interface R extends RegExp {
}
```

![](https://cdn.wallleap.cn/img/pic/illustration/202309111024049.png)

#### 区别

区别 1：interface 只**描述对象**，type 则描述所有数据

区别 2：type 只是**别名**，interface 则是类型声明

区别 3：type 不可重新赋值，interface 可以（会合并）；对外 API 尽量使用 interface，方便扩展，对内 API 尽量用 type，防止代码分散（例如 React 中 Props）

```ts
type A = number
// A = number | string // 这样是不可以的
// 好处是可以提升计算机效率
// 缺点是不好扩展
```

interface 可以直接进行合并

```ts
interface X {
  name: string
}
interface X {
  age: number
}

const x: X = {
  name: 'Tom',
  age: 18
}
```

interface 如何扩展，举个例子

```ts
import { AxiosRequestConfig } from 'axios'
declare module 'axios' { // 仅在这个模块生效
  export interface AxiosRequestConfig {
    _autoLoading?: boolean
    _mock?: string
  }
}
```

扩展 String

```ts
declare global { // 全局
  interface String {
    padZero(x:string): void
  }
}

const s = 'hello'
s.padZero('hello')
```

type 扩展/继承

```ts
type A = { aaa: string }
type B = { bbb: string } & A

const b: B = {
  aaa: 'yes',
  bbb: 'yes'
}
```
