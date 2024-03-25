---
title: 001-了解 ArkTS 语言
date: 2023-12-11 14:48
updated: 2023-12-12 09:41
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

## TypeScript

ArkTS 基于 TypeScript

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

条件控制

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

循环

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
  console.log(names[i])
}
for(const name of names) {
  console.log(name)
}
```

函数

```ts
function print(name: string): void {
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

function print(name?: string) {
  name = name ? name : 'nick'
  console.log(name)
}
print()

function print(name: string = 'nick') {
  console.log(name)
}
print()
```

类和接口

```ts
enum Msg{
  Left: 'left',
  Right: 'right
  '
}

interface A {
  go(msg: Msg): void
}

class B implements A {
  go(msg: Msg): void {
    console.log('go' + msg)
  }
}

let a: A = new B()
a.go(Msg.Left)
```
