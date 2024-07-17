---
title: 001-了解 ArkTS 语言
date: 2023-12-11 14:48
updated: 2024-06-17 10:50
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - 001-了解 ArkTS 语言
rating: 1
tags:
  - 鸿蒙
category: 鸿蒙
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
description: 文章描述
source: 
url: //myblog.wallleap.cn/post/1
---

网页制作一个按钮，HTML 控制页面元素，CSS 控制布局和样式，JS 控制页面逻辑和数据状态

```html
<!-- HTML -->
<button>点击0次</button>
<!-- CSS -->
<style>
body {
  text-align: center;
}
button {
  background: #36D;
  color: #fff;
  border-radius: 2px;
}
</style>
<!-- JavaScript -->
<script>
  const btn = document.querySelector('button')
  let i = 1
  btn.onclick = () => btn.innerHTML = `点击${i++}次`
</script>
```

![](https://cdn.wallleap.cn/img/pic/illustration/202312120941938.png)

新建鸿蒙项目可以看到语言是 ArkTS

## ArkTS

- 声明式 UI
- 状态管理

```ts
build() {
  Button('点击0次')
    .backGroundColor('#36D')
}
```

想要什么只需要在 build 中写什么

```ts
build() {
  Row() {
    Button('点击0次')
      .backGroundColor('#36D')
  }
  .width('100%')
  .justifyContent(FlexAlign.Center)
}
```

定义变量

```ts
times: number = 0
build() {
  Row() {
    Button(`点击${this.times}次`)
      .backGroundColor('#36D')
      .onClick(() => this.times++)
  }
  .width('100%')
  .justifyContent(FlexAlign.Center)
}
```

状态管理

```ts
@State times: number = 0
build() {
  Row() {
    Button(`点击${this.times}次`)
      .backGroundColor('#36D')
      .onClick(() => this.times++)
  }
  .width('100%')
  .justifyContent(FlexAlign.Center)
}
```

整体

```ts
// 装饰器：用来装饰类结构、方法、变量
@Entry // 标记当前组件是入口组件，可以单独访问，如果标有标记就是普通组件，需要使用其他入口组件引用才能显示
@Component // 标记自定义组件
// 所有内容写在一个结构体中
struct Index { // 自定义组件：可复用的 UI 单元，可以封装多个，到其他地方使用
  @State times: number = 0 // 标记该变量是状态变量，值变量的时候会触发 UI 刷新

  build() { // UI 描述：内部以声明式方式描述 UI 结构
    // 内置组件：容器组件（完成页面布局）和基础组件（自带样式和功能的页面元素）
    Row() {
      Column() {
        Text(`${this.times}`)
          .fontSize(50) // 属性方法：设置组件的 UI 样式
          .fontWeight(FontWeight.Bold)
        Button('点击我')
          .padding(10)
          .width(100)
          .margin(20)
          .backgroundColor('#ffa107a1')
          .onClick((e) => { // 事件方法：设置组件的事件回调
            this.times++ // 处理事件
          })
      }
      .width('100%')
    }
    .height('100%')
  }
}
```

- 装饰器： 用于装饰类、结构、方法以及变量，并赋予其特殊的含义。如上述示例中@Entry、@Component 和@State 都是装饰器，[@Component](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V2/arkts-create-custom-components-0000001473537046-V2#section1430055924816) 表示自定义组件，[@Entry](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V2/arkts-create-custom-components-0000001473537046-V2#section1430055924816) 表示该自定义组件为入口组件，[@State](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V2/arkts-state-0000001474017162-V2) 表示组件中的状态变量，状态变量变化会触发 UI 刷新。
- [UI描述](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V2/arkts-declarative-ui-description-0000001524416537-V2)：以声明式的方式来描述 UI 的结构，例如 build() 方法中的代码块。
- [自定义组件](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V2/arkts-create-custom-components-0000001473537046-V2)：可复用的 UI 单元，可组合其他组件，如上述被@Component 装饰的 struct Hello。
- 系统组件：ArkUI 框架中默认内置的基础和容器组件，可直接被开发者调用，比如示例中的 Column、Text、Divider、Button。
- 属性方法：组件可以通过链式调用配置多项属性，如 fontSize()、width()、height()、backgroundColor() 等。
- 事件方法：组件可以通过链式调用设置多个事件的响应逻辑，如跟随在 Button 后面的 onClick()。
- 系统组件、属性方法、事件方法具体使用可参考 [基于ArkTS的声明式开发范式](https://developer.huawei.com/consumer/cn/doc/harmonyos-references-V2/ts-components-summary-0000001478181369-V2)。

除此之外，ArkTS 扩展了多种语法范式来使开发更加便捷：

- [@Builder](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V2/arkts-builder-0000001524176981-V2)/[@BuilderParam](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V2/arkts-builderparam-0000001524416541-V2)：特殊的封装 UI 描述的方法，细粒度的封装和复用 UI 描述。
- [@Extend](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V2/arkts-extend-0000001473696678-V2)/[@Styles](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V2/arkts-style-0000001473856690-V2)：扩展内置组件和封装属性样式，更灵活地组合内置组件。
- [stateStyles](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V2/arkts-statestyles-0000001482592098-V2)：多态样式，可以依据组件的内部状态的不同，设置不同样式。

## TypeScript

ArkTS 基于 TypeScript

<https://typescriptlang.org>

TS 是基于 JS 的（静态类型定义）

```ts
// string、number、boolean
let msg: string = 'hello world'
let age: number = 21
let finished: boolean = true

// any 不确定类型 可以是任意类型
let a: any = 'Jack'
a = 1

// 联合类型 union
let u: string|number|boolean = 'rose'
u = 2

// Object 对象
const p = { name: 'Jack', age: 21 }
console.log(p.name) // p['name']

// Array 数组
const names: Array<string> = ['Jack', 'Rose']
const ages: number[] = [21, 18]
console.log(names[0])
```

**条件控制**

- if-else
- switch

```ts
let num: number = 21
if(num % 2 === 0) {
  console.log('偶数')
} else {
  console.log('奇数')
}
```

空字符串 `''`、数字 `0`、`null`、`undefined` 都被认为是 `false`，其他值为 `true`

**循环**

- for
- while
- 数组
	- for-in
	- for-of

```ts
for(let i = 1; i <= 10; i++) {
  console.log(i)
}

let i = 1
while(i <= 10) {
  console.log(i++)
}

let names: string[] = ['Jack', 'Rose']
for(const i in names) {
  console.log(names[i]) // 角标
}
for(const name of names) {
  console.log(name) // 元素
}
```

**函数**

```ts
function print(name: string): void { // 无返回值 void，可省略
  console.log(name)
}
print('Jack')

function sum(x: number, y: number): number {
  return x + y
}
console.log(sum(21 + 18))

const print = (name: string) => {
  console.log(name)
}
print('Jack')

function print(name?: string) { // 可传可不传，? 代表可选
  name = name ? name : 'nick'
  console.log(name)
}
print()

function print(name: string = 'nick') { // 参数默认值，没传就用默认值
  console.log(name)
}
print()
```

**类和接口**

```ts
// 枚举
enum Msg {
  Left: 'left',
  Right: 'right'
}

// 定义接口
interface A {
  go(msg: Msg): void
}
// 实现接口
class B implements A {
  go(msg: Msg): void {
    console.log('go' + msg)
  }
}
// 初始化对象
let a: A = new B()
// 调用方法
a.go(Msg.Left)

// 定义矩形类
class Rectangle {
  // 成员变量，加 private 代表私有
  private width: number
  private length: number
  // 构造函数
  constructor(width: number, length: number) {
    this.width = width
    this.length = length
  }
  // 成员方法
  public area(): number {
    return this.width * this.length
  }
}

// 定义正方形
class Square extends Rectangle {
  constructor(side: number) {
    // 调用父类构造
    super(side, side)
  }
}

const s = new Square(10)
console.log('正方形的面积为' + s.area(10))
```

**模块开发**

每个文件都是一个模块（module）

```ts
// reactangle.ts
export class Rectangle {
  public width: number
  public length: number
  // 构造函数
  constructor(width: number, length: number) {
    this.width = width
    this.length = length
  }
}
export function area(rec: Rectangle): number {
  return rec.width * rec.lengt
}

// main.ts
import {Rectangle, area} from './Rectangle'
const r = new Rectangle(10, 20)
console.log(area(r))
```
