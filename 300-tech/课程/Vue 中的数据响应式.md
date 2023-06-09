---
title: Vue 中的数据响应式
date: 2023-06-05 16:00
updated: 2023-06-07 10:36
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - Vue 中的数据响应式
rating: 1
tags:
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

为了更好地理解后面的进阶属性，需要对 `data` 有个更深入的了解

文档在这里 [深入响应式原理](https://v2.cn.vuejs.org/v2/guide/reactivity.html)

当你把一个普通的 JavaScript 对象传入 Vue 实例作为 `data` 选项，Vue 将遍历此对象所有的 property，并使用 [`Object.defineProperty`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty) 把这些 property 全部转为 [getter/setter](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Working_with_Objects#%E5%AE%9A%E4%B9%89_getters_%E4%B8%8E_setters)。

## 预备知识

上面给了 MDN 的链接，可以自己看，也可以直接看这里

### ES 6 语法 setter 和 getter

我们定义了一个 person 对象，需要获取姓名和设置姓名，因此写下了如下代码：

```js
const person = {
	firstName: "Wang",
	lastName: "Lu",
	getName() {
		return this.lastName + this.firstName;
	},
	setName(fullName) {
		this.lastName = fullName[0];
		this.firstName = fullName.slice(1);
	}
};
console.log(person.getName());
person.setName("张", "杰");
console.log(person.getName());
```

现在有个需求，我在获取 name 的时候不想要后面的括号，就像这样 `person.name`

使用 ES 6 语法 getter，一个 getter 是一个获取某个**特定属性的值**的方法

```js
const person1 = {
  firstName: "Wang",
  lastName: "Lu",
  get name() {
    return this.lastName + this.firstName;
  }
}
console.log(person1.name);  // 不加括号的函数（计算属性）
```

需求二：设置的时候也不加括号，直接 `person.name = 'xxx'`

一个 setter 是一个设定某个**属性的值**的方法

```js
const person1 = {
	firstName: "Wang",
	lastName: "Lu",
	get name() {
		return this.lastName + this.firstName;
	},
	set name(fullName) {
		this.lastName = fullName[0];
		this.firstName = fullName.slice(1);
	}
};
console.log(person1.name);
person1.name = "张杰";
console.log(person1.name);
```

在初始化对象的时候，到函数前面加关键词 `get` 或 `set`，后面调用或赋值的时候像正常的属性值使用（给 getter 赋值会以参数形式传入到函数中，所以只能有一个参数）

不能为同一个属性设置多个 set，例如一下都是不可以的

```js
// 1
set x(v) {}
set x(v) {}
// 2
x: xxx
set x(v) {}
```

### ES 6 新增的 Object.defineProperty()

上面的都是直接在对象初始化的时候设置 getter 和 setter，那对于已经定义好的对象，需要怎么添加 getter 和 setter 呢

`Object.defineProperty()` 静态方法会直接在一个对象上定义一个新属性，或修改其现有属性，并返回此对象

```js
let _xxx = 0
const obj = {
  name: 'luwang',
  age: '19'
}
Object.defineProperty(obj, 'xxx', {
  get() {
    return _xxx  // 需要返回，只能在外面声明
  },
  set(value) {
    _xxx = value
  },
})
```

三个参数

- 第一个：要定义属性的对象
- 第二个：一个字符串或 [`Symbol`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol)，指定了要定义或修改的属性键
- 第三个：要定义或修改的属性的描述符，是个对象，里面属性有
	- `configurable`：默认值为 `false`
	- `enumerable`：默认值为 `false`
	- `value`：属性值，默认值为 `undefined`
	- `writable`：默认值为 `false`
	- `get`、`set` 这两个是**访问描述符**，不能和上面两个**数据描述符**同时使用

## 看一下 data

```js
const myData = {
  n: 0
}
console.log(myData)

const vm = new Vue({
  data: myData,
  template: `<div>{{this.n}}<button @click=add>+10</button></div>`,
  methods: {
    add() {
      this.n += 10  // vm.n
      // myData.n += 10
    }
  }
}).$mount('#app')

setTimeout(() => {
  myData.n += 10
  console.log(myData)
  console.log(vm)
}, 3000)
```

在控制台中打印，一开始是 `{n:0}`，穿给 `new Vue` 之后立马变成 `{n: (...)}` / `...Obsert`

`{n: (...)}` 中有 get、set 等

## 代理和监听

之前我们定义 data 中的数据都是直接初始化的时候定义的

```js
let data0 = {
  n: 0
}
```

需求一：现在不在初始化的时候定义，用 `Object.defineProperty` 定义 n

```js
let data1 = {}
Object.defineProperty(data1, 'n', {
  value: 0
})
console.log(data1.n)  // 0
```

需求二：让 n 不能小于 0

```js
let data2 = {}
data2._n = 0  // 随便放在哪，只是现在放在这更方便
Object.defineProperty(data2, 'n', {
  get() {
    return this._n  // data2._n
  },
  set(value) {
    if(value < 0) return  // 小于 0 直接白写
    this._n = value
  }
})
console.log(data2.n)  // 0
data2.n = -1
console.log(data2.n)  // 0
data2.n = 1
console.log(data2.n)  // 1
```

可以直接修改 `data2._n`，例如 `data2._n = -1`

需求三：使用代理，不在对象上暴露任何可访问的属性，甚至不设置对象名

```js
let data3 = proxy({ data: {n: 0}})  // 匿名对象，无法访问
function proxy({data}) {  // 解构赋值
  const obj = {}
  for(const key in data) {
    Object.defineProperty(obj, key, {
      get() {
        return data[key]
      },
      set(value) {
        if(value < 0) return
        data[key] = value
      }
    })
  }
  return obj  // obj 就是代理
}
// data3 就是 obj
console.log(data3.n)  // 0
data3.n = -1
console.log(data3.n)  // 0
data3.n = 1
console.log(data3.n)  // 1
```

需求四：我非得给个名字

```js
let myData4 = {n: 0}
let data4 = proxy({data: myData4})
console.log(data4.n)  // 0
myData4.n = -1
console.log(data4.n)  // -1
```

这样也能修改 n

需求五：修改 myData 也会被拦截

```js
let myData5 = {n:0}
let data5 = proxy2({data:myData5})
function proxy2({data}) {
  const obj = {}
  for(const key in data) {
    let value = data[key]
    delete data[key]  // 可以不写，声明新的会覆盖旧的
    Object.defineProperty(data, key, {
      get() {
        return value
      },
      set(newValue) {
        if(newValue < 0) {
          console.log('changed failed')
          return
        }
        value = newValue
      }
    })
    // 加了上面的就会监听 data
    Object.defineProperty(obj, key, {
      get() {
        return data[key]      
      },
      set(value) {
        if(value < 0) return
        data[key] = value
      }
    })
  }
  return obj
}
myData5.n = -1  // changed failed
console.log(data5.n)  // 0
```

现在看下 `const data5 = proxy2({ data: myData5})` 是不是有点眼熟了（`const vm = new Vue({data: myData})`）

总结：

- `Object.defineProperty`
	- 可以给对象添加属性值 `value`（甚至其他是否可写等配置项）
	- 可以给对象添加 getter / setter
	- getter / setter 用于对属性的读写进行监控
- 代理（一种设计模式）
	- 就像房东和中介
	- 对 myData 对象的属性读写，全权由另一个对象 `vm` 负责，那么 `vm` 就是 myData 的代理
	- 比如 `myData.n` 不用，偏要用 `vm.n` 来操作 `myData.n`
- `vm = new Vue({data: myData})`
	1. 会让 vm 成为 myData 的代理（proxy）（Vue() 里面 this 就是 vm）
	2. 会对 myData 的所有属性进行监控
		- 为什么要监控：为了防止 myData 的属性变了，而 vm 不知道
		- vm 知道了又怎么样：知道属性变了就可以调用 `render(data)`，进行视图层面的更新

![示意图](https://cdn.wallleap.cn/img/pic/illustration/202306070051804.webp?imageMogr2/format/webp/interlace/1/quality/80%7Cwatermark/1/image/aHR0cDovL3dhbGxsZWFwLTEyNTkwODQzMzAuY29zLmFwLWd1YW5nemhvdS5teXFjbG91ZC5jb20vaW1nL3dhdGVybWFyazEucG5n/gravity/southeast/dx/16/dy/16/scatype/3/spcent/10/dissolve/90%7Cwatermark/3/type/3/text/bHV3YW5nMHdhbGxsZWFw)

n 是覆盖掉的（一直在对 n 进行修改/代理）

同理，Vue 对 methods、computed 也有处理

如果 data 有多个属性 n、m、k，那么就会有 get n / get m / get k 等

## 数据响应式

响应式

- 例如我打你一拳，你会喊疼，那你就是相应式的
- 如果一个物体能够对外界的刺激做出反应，它就是相应式的

Vue 的 data 是响应式的

- `const vm = new Vue({data: {n: 0}})`，如果修改 `vm.n` ，那么 UI 中的 n 就会响应修改
- Vue 2 通过 `Object.defineProperty` 来实现数据响应式

响应式网页

- 如果改变窗口大小，网页内容会做出相应，那就是响应式网页
- 例如 <https://www.smashingmagazine.com>

## Vue 的 data 有 bug

`Object.defineProperty(obj, 'n', {})` 的问题

- 必须要有一个 `'n'`，才能监听和代理 `obj.n`
- 如果开发者并没有给 `'n'`

```js
new Vue({
  data: {}, // 空的
  template: `<div>{{n}}</div>`
}).$mount('#app')
```

页面上不显示，控制台输出红色警告，n 没有定义

```js
new Vue({
  data: {
    obj: {
      a: 0  // obj、obj.a 会被 Vue 监听和代理
    },
  }
  template: `<div>{{obj.b}}<button @click="setB">set b</button></div>`,
  methods: {
    setB() {
      this.obj.b = 1  // obj.b 不会被监听和代理
    }
  }
}).$mount(`#app`)
```

`obj.b` 不会在页面上显示，且不会报错或警告（Vue **只会检查第一层**属性）

同时改 `obj.a` 和 `obj.b`，都能改，而且都会显示

Vue 没办法监听一开始不存在的 `obj.b`

解决办法：

- 先在 obj 对象中声明 b，例如 `obj: {..., b: undefined}`
- 使用 Vue 提供的 `Vue.set` 或 `vm.$set`
	1. 新增 key
	2. 自动创建代理和监听（如果没有创建过）
	3. 触发 UI 更新（但并不会立刻更新）

```js
setB() {
  Vue.set(this.obj, 'b', 1)  // 和 ES 的 set 不同
  // vm.$set(this.obj, 'b', 1) // vm.$set === this.$set === Vue.set
}
changeB() {
  this.obj.b += 10
}
```

第一次需要 set，后面再修改就会响应了

## Vue 中数组的变异方法

```js
new Vue({
  data: {
    arr: ['a', 'b', 'c']
  },
  template: `
    <div>{{arr}}<button @click="setD">set d</button></div>
  `,
  methods: {
    setD() {
      this.arr[3] = 'd'
    }
  }
}).$mount('#app')
```

点击按钮并没有反应，用 set 试试

```js
setD() {
  this.$set(this.arr, 3, 'd')
}
```

但是数组长度很难预测，不能提前声明，用 set 也不太合适，使用 push 试试

```js
setD() {
  this.arr.push('d')
  console.log(this.arr)
}
```

这样可以实现，而且打印出来的原型中和 JS 中的不一样，下一层的原型才是 Array 的方法

篡改了数组的方法，在文档 [列表渲染 — 数组更新检测)](https://v2.cn.vuejs.org/v2/guide/list.html#%E6%95%B0%E7%BB%84%E6%9B%B4%E6%96%B0%E6%A3%80%E6%B5%8B) 中有写，变更了以下方法：`push`、`pop`、`shift`、`unshift`、`splice`、`sort`、`reverse`，它们也会触发视图更新

怎么篡改的？

ES 6 实现一下——继承

```js
class VueArray extends Array {
  push(...args) {
    const oldLength = this.length  // this 是当前数组
    // 先调用父类上的 push
    super.push(...args)
    console.log(' pushed')
    for(let i = oldLength; i < this.length; i++) {
      // 每一个 set
      Vue.set(this, i, this[i])
    }
  }
}
const a = new VueArray(1,2,3,4)
console.log(a)
a.push(5)
console.log(a)
```

ES 5 怎么实现——原型

```js
const vueArrayPrototype = {
  push: funtion() {
    console.log('pushed')
    return Array.prototype.push.apply(this, arguments)
  }
}
vueArrayPrototype.__proto__ = Array.prototype
const a = Object.create(vueArrayPrototype)
a.push(1)
```

总结：

对象中新增的 key

- Vue 没有办法实现监听和代理
- 要使用 set 来新增 key，创建监听和代理，更新 UI
- 最好提前把属性都写出来，不要新增 key
- 但数组做不到 “不新增 key”

数组中新增的 key

- 也可以用 set 来新增 key，且创建监听和代理，更新 UI
- 不过尤雨溪篡改了 7 个 API 方便对数组进行增删
- 这 7 个 API 会自动处理监听和代理，并更新 UI
