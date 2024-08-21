---
title: Vue 源码之指令、双向绑定和 watch
date: 2023-08-04T10:20:00+08:00
updated: 2024-08-21T10:32:34+08:00
dg-publish: false
aliases:
  - Vue 源码之指令、双向绑定和 watch
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
  - Vue
  - web
url: //myblog.wallleap.cn/post/1
---

这是我参与更文挑战的第 10 天，活动详情查看： [更文挑战](https://juejin.cn/post/6967194882926444557?utm_campaign=30day&utm_medium=Ccenter&utm_source=20210528 "https://juejin.cn/post/6967194882926444557?utm_campaign=30day&utm_medium=Ccenter&utm_source=20210528")

> 本文是关于 Vue 源码的学习笔记，这里做个总结与分享，有不足之处还望斧正~

## 学习目标

自己手写一个简单的 Vue 库，实现对双大括号的识别，数据的双向绑定（对 `v-model` 指令的处理）以及对 `watch` 的实现

## 手写实现

### Vue 类的创建

当我们 `new` 一个 Vue 实例时，将处理下面 6 件事（简单版）

1. 把用户定义的传给 Vue 实例的参数 options 对象赋值给实例本身，方便用户获取
2. 把 options 里的 `data` 添加到 Vue 实例的 `_data` 属性上，方便后续操作
3. 把 `data` 的数据添加到 Vue 实例上，方便用户获取
4. 把 `data` 的数据改成响应式的
5. 调用 `_initWatch()` 处理 options 里用户定义在 `watch` 对象里的内容（同理，定义在生命周期函数里内容，比如 `created`，也是在 constructor 进行初始化处理）
6. 对传入的 el 内容进行编译

> 注意，下面文件中 import 的 observe 和 Watcher 为之前的内容，
> 详见 [《数据响应式原理 - 01》](https://juejin.cn/post/6969741580412387336 "https://juejin.cn/post/6969741580412387336")，[《数据响应式原理 - 02》](https://juejin.cn/post/6970853854367875103 "https://juejin.cn/post/6970853854367875103")，[《数据响应式原理 - 03》](https://juejin.cn/post/6971596170921508900 "https://juejin.cn/post/6971596170921508900")

```js
// Vue.js
import Compile from './Compile.js'
import observe from './observe.js'
import Watcher from './Watcher.js'

export default class Vue {
  constructor(options) {
    this.$options = options || {} // 让创建 Vue 实例的用户能拿到 options
    this._data = options.data || undefined
    // 初始化定义在 options 上的 data
    this._initData()
    // 数据变为响应式数据
    observe(this._data)
    // 对 options 里用户定义的 watch 进行处理
    this._initWatch() 
    // 模板编译
    new Compile(options.el, this)
  }

  // 将 data 数据定义到 vue 实例上，这样在 index.html 里就能直接通过 vm.xx 获取属性
  _initData() {
    Object.keys(this._data).forEach(item => {
      Object.defineProperty(this, item, {
        configurable: true,
        enumerable: true,
        get() {
          return this._data[item]
        },
        set(newVal) {
          this._data[item] = newVal
        }
      })
    })
  }

  _initWatch() {
    const watch = this.$options.watch
    Object.keys(watch).forEach(item => {
      // 这里第一个参数可以用 this 也就是 vue 实例
      // 是因为 _initData 已经把所有 data 里的数据定义到 vue 实例上了
      new Watcher(this, item, watch[item])
    })
  }
}
```

### Fragment 的生成

实际上，vue 是借助 AST 进行这部分操作的

> 关于 AST 的介绍，详见 [《AST 抽象语法树》](https://juejin.cn/post/6975414784992739359 "https://juejin.cn/post/6975414784992739359")

这里为了方便，我们借助文档片段（DocumentFragment）进行编译处理，这样比直接操作真实 DOM 性能更好。代码如下，将传入的 el 的节点都放到文档片段中。

```js
node2Fragment(el) {
  const fragment = document.createDocumentFragment()
  let child
  while (child = el.firstChild) {
    fragment.appendChild(child)
  }
  return fragment
}
```

之后就是对文档片段的编译，获取到字符串模板（`el` 里的内容）的所有子节点，通过 `nodeType` 判断是元素节点（调用 `compileElement` 处理）还是文本节点，若是文本节点，就看看是不是双大括号里的内容，若是则调用 `compileText` 处理

```js
compile(el) {
  const nodeList = el.childNodes // 这里得到是一个 NodeList 类型的集合，不是数组，但可以使用 forEach() 来迭代
  const regExp = /\{\{(.*)\}\}/ // 正则获取 mustache 内容
  nodeList.forEach(item => {
    const text = item.textContent // 获取到的 item 都是节点，不是字符串，所以获取文字要用 .textContent
    if (item.nodeType === 1) { // 元素节点
      this.compileElement(item)
    } else if (item.nodeType === 3 && regExp.test(text)) { // 文本节点
      const dataName = text.match(regExp)[1].trim()
      this.compileText(item, dataName)
    }
  })
}
```

## v-model 的实现

`v-model` 一般是绑定在 input 上的，对于 `v-model` 的处理在 `compileElement` 方法里完成，也就是编译元素节点的时候，先通过 `node.attributes` 收集该元素节点身上的所有属性，转成数组后遍历，看看属性名是不是以 `v-` 开头的，是的话说明这个属性是一个指令。再看看 `v-` 后面是否为 `model`，若是，则对这个属性的值添加 watcher，并且去 `data` 里寻找对应的 value，赋值给这个 input。监听用户更改 input 的值的行为，实时更改 `data` 里的属性值。

综上整理 Compile.js 文件如下

```js
// Compile.js
import Watcher from './Watcher.js'

export default class Compile {
  constructor(el, vue) {
    this.$vue = vue // vue 实例
    this.$el = document.querySelector(el) // 挂载点
    // 如果用户传了挂载点
    if (this.$el) {
      // 调用函数，让节点变为 fragment，类似 mustache 里的 tokens
      // vue 源码里实际用的是 ast
      const $fragment = this.node2Fragment(this.$el)
      // 编译，对文档片段(fragment) 进行编译，这样比直接编译真实的 DOM 节点(this.$el) 性能更好
      this.compile($fragment)
      // 将文档片段上 DOM 树
      this.$el.appendChild($fragment)
    }
  }
  node2Fragment(el) {
    const fragment = document.createDocumentFragment()
    let child
    while (child = el.firstChild) {
      fragment.appendChild(child)
    }
    return fragment
  }

  compile(el) {
    const nodeList = el.childNodes // 这里得到是一个 NodeList 类型的集合，不是数组，但可以使用 forEach() 来迭代
    const regExp = /\{\{(.*)\}\}/ // 正则获取 mustache 内容
    nodeList.forEach(item => {
      const text = item.textContent // 获取到的 item 都是节点，不是字符串，所以获取文字要用 .textContent
      if (item.nodeType === 1) { // 元素节点
        this.compileElement(item)
      } else if (item.nodeType === 3 && regExp.test(text)) { // 文本节点
        const dataName = text.match(regExp)[1].trim()
        this.compileText(item, dataName)
      }
    })
  }

  compileElement(node) {
    const attrs = node.attributes // 这里得到的 attrs 是一个 NamedNodeMap 对象，不是数组
    // Array.prototype.slice.call(attrs) 将 attrs 转为数组
    // slice 返回一个新的数组对象，是一个原数组的浅拷贝；通过 call 让 attrs 能使用 Array.prototype 的 slice 方法
    Array.prototype.slice.call(attrs).forEach(item => {
      // 获取 name 和 value
      const name = item.name 
      const value = item.value
      if (name.indexOf('v-') === 0) { // 以 v- 开头的属性即为指令
        const directiveName = name.substring(2)
        if (directiveName === 'model') {
          // 添加 watcher，一旦改变了 v-model 绑定的这个属性的值，就能实时响应
          new Watcher(this.$vue, value, newVal => {
            node.value = newVal
          })
          // 获取这个 value 在 data 里的值
          const v = this.getDataValue(this.$vue, value)
          // 这里仅处理这个 node 是 input 这种情况，所以直接用 value 属性赋值
          node.value = v
          // 如果输入 input 的值改变
          node.addEventListener('input', e => {
            // 将 data 里的属性的值改成输入 input 的值
            this.setDataValue(this.$vue, value, e.target.value)
          })
        }
      }
    })
  }

  compileText(node, dataName) {
    // 获取到 data 中变量的值，然后放到对应节点里
    node.textContent = this.getDataValue(this.$vue ,dataName)
    new Watcher(this.$vue, dataName, newVal => {
      node.textContent = newVal
    })
  }

  getDataValue(obj, dataName) {
    const nameArr = dataName.split('.')
    const value = nameArr.reduce((acc, cur) => {
      return acc[cur]
    }, obj)
    return value
  }
  
  // 设置 data 里某个属性的值
  setDataValue(obj, dataName, dataValue) {
    const nameArr = dataName.split('.')
    let val = obj
    nameArr.forEach((item, index, arr) => {
      if (index < arr.length - 1) {
        val = val[item]
      } else {
        val[item] = dataValue
      }
    })
  }
}
```

## One More Thing

### DocumentFragment 介绍

DocumentFragments 是 DOM 节点。它们不是主 DOM 树的一部分。通常的用例是创建文档片段，将元素附加到文档片段，然后将文档片段附加到 DOM 树。在 DOM 树中，文档片段被其所有的子元素所代替。

因为文档片段存在于内存中，并不在 DOM 树中，所以将子元素插入到文档片段时不会引起页面回流（对元素位置和几何上的计算）。因此，使用文档片段通常会带来更好的性能。
