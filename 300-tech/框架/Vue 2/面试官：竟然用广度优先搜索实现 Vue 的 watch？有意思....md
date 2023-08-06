---
title: 面试官：竟然用广度优先搜索实现 Vue 的 watch？有意思...
date: 2023-08-04 10:32
updated: 2023-08-04 10:52
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - 面试官：竟然用广度优先搜索实现 Vue 的 watch？有意思...
rating: 1
tags:
  - Vue
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

## 前言

通过前面几篇文章，我们对 Vue3 中的响应式设计有了初步的了解。

1. [面试官：Vue3响应式系统都不会写，还敢说精通？](https://juejin.cn/post/7090328834318270494 "https://juejin.cn/post/7090328834318270494")
2. [面试官：你觉得Vue的响应式系统仅仅是一个Proxy？](https://juejin.cn/post/7251974224923050021 "https://juejin.cn/post/7251974224923050021")
3. [Vue3：原来你是这样的“异步更新”](https://juejin.cn/post/7255247517389635639 "https://juejin.cn/post/7255247517389635639")
4. [为啥面试官总喜欢问computed是咋实现的?](https://juejin.cn/post/7259405321020424251 "https://juejin.cn/post/7259405321020424251")

这一篇我们试着实现一个 `watch`

## 1.# 两种 watch 的基本用法

### 1.1# 通过函数回调监听数据

最基本的用法是给 `watch` 指定一个回调函数并返回你想要监听的**响应式数据**。

```js
const state1 = reactive({
  name: '前端胖头鱼',
  age: 100
})

watch(() => state1.age, () => {
  console.log('state1的age发生变化了', state1.age)
})

state1.age = 200

setTimeout(() => {
  state1.age = 300
}, 500)
```

![](https://cdn.wallleap.cn/img/pic/illustration/202308041032634.jpeg)

### 1.2# 直接监听一个对象

还可以直接监听一个响应式对象来观测它的变化。

```js
const state1 = reactive({
  name: '前端胖头鱼',
  age: 100,
  children: {
    name: '胖小鱼',
    age: 10
  }
})

watch(state1, () => {
  console.log('state1发生变化了', state1)
})

state1.age = 200

setTimeout(() => {
  state1.children.age = 100
}, 500)

```

![](https://cdn.wallleap.cn/img/pic/illustration/202308041032636.jpeg)

## 2.# 实现 watch 最核心的点

其实 `watch` 的底层实现非常简单，和 `computed` 一样都需要借助 `任务调度`。

简单来说就是**感知数据的变化**，数据发生了变化就执行对应的回调，**那么怎么感知呢？**

```js
const state = reactive({
  name: '前端胖头鱼'
})

useEffect(() => {
  // 原本state发生变化之后，应该执行这里
  console.log(state.name)
}, {
  // 但是指定scheduler之后，会执行这里
  scheduler () {
    console.log('state变化了')
  }
})

state.name = '胖小鱼'
```

聪明的你肯定也猜到了，**scheduler**不就是天然**感知数据的变化**的工具吗？

没错，`watch` 的实现少不了它，来吧，搞起！！！

## 3.# 支持两种使用方式

### 3.1 支持回调函数形式

```js
const watch = (source, cb) => {
  effect(source, {
    scheduler () {
      cb()
    },
  })
}

// 测试一波
const state = reactive({
  name: '前端胖头鱼',
})

watch(() => state.name, () => {
  console.log('state.name发生了变化', state.name)
})

state.name = '胖小鱼'

```

![](https://cdn.wallleap.cn/img/pic/illustration/202308041032637.jpeg)

### 3.2 支持直接传递响应式对象

不错哦！第一种方式已经初步实现了，接下来搞第二种。

第二种直接传入**响应式对象**的方式和第一种传入**回调函数并指向响应式数据**的区别是什么？

在于我们需要手动遍历这个**响应式对象**使得它的任意属性发生变化我们都能感知到。

### 3.3 广度优先搜索遍历深层嵌套的属性

此时就到了这篇文章 `装逼(额~~)` 的点了。如果想访问一个深层嵌套对象的所有属性，最常见的做法就是递归。

如果你想在面试的过程中秀一波，我觉得使用广度优先搜索是个不错的主意（狗头脸😄），代码也非常简单，就不详细解释了。

`如果您对广度优先搜索和深度优先搜索感兴趣欢迎在评论区留言，我会单独写一篇文章来讲它。`

```js
 const bfs = (obj, callback) => {
  const queue = [ obj ]

  while (queue.length) {
    const top = queue.shift()

    if (top && typeof top === 'object') {
      for (let key in top) {
        // 读取操作出发getter，完成依赖搜集
        queue.push(top[ key ])
      }
    } else {
      callback && callback(top)
    }
  }
}

const obj = {
  name: '前端胖头鱼',
  age: 100,
  obj2: {
    name: '胖小鱼',
    age: 10,
    obj3: {
      name: '胖小小鱼',
      age: 1,
    }
  },
}

bfs(obj, (value) => {
  console.log(value)
})
```

![](https://cdn.wallleap.cn/img/pic/illustration/202308041032638.jpeg)

我们已经能够读取深层嵌套对象的任意属性了，接下来继续完善 `watch` 方法

```js
const watch = (source, cb) => {
  let getter
  // 处理传回调的方式
  if (typeof source === "function") {
    getter = source
  } else {
    // 封装成读取source对象的函数，触发任意一个属性的getter，进而搜集依赖
    getter = () => bfs(source)
  }

  const effectFn = effect(getter, {
    scheduler() {
      cb()
    }
  })
}

// 测试一波
const state = reactive({
  name: "前端胖头鱼",
  age: 100,
  obj2: {
    name: "胖小鱼",
    age: 10,
  },
})

watch(state, () => {
  console.log("state发生变化了");
});
```

![](https://cdn.wallleap.cn/img/pic/illustration/202308041032639.jpeg)

看来还是有不少坑啊！虽然我们实现了 n 层嵌套对象属性的读取（理论上所有的属性改变都应该触发回调），但是 `state.obj2.name = 'yyyy'` 却没有被感知到，为什么呢？

### 3.4 # 浅响应与深响应

回顾一下 `reactive` 函数，你会发现，当 `value` 本身也是一个对象的时候，我们并不会使 `value` 也变成一个响应式数据。

所以哪怕我们通过 `bfs` 方法遍历了该对象的所有属性，也仅仅是第一层的 `key` 具有了响应式效果而已。

```js
// 统一对外暴露响应式函数
function reactive(state) {
  return new Proxy(state, {
    get(target, key) {
      const value = target[key]
      // 搜集key的依赖
      // 如果value本身是一个对象，对象下的属性将不具有响应式
      track(target, key) 

      return value;
    },
    set(target, key, newValue) {
      // console.log(`set ${key}: ${newValue}`)
      // 设置属性值
      target[key] = newValue

      trigger(target, key)
    },
  })
}

```

解决办法也很简单，是对象的情况下再给他 `reactive` 一次就好了。

```js
// 统一对外暴露响应式函数
function reactive(state) {
  return new Proxy(state, {
    get(target, key) {
      const value = target[key]
      // 搜集key的依赖
      // 如果value本身是一个对象，对象下的属性将不具有响应式
      track(target, key) 
      // 如果是对象，再使其也变成一个响应式数据
      if (typeof value === "object" && value !== null) {
        return reactive(value);
      }

      return value;
    },
    set(target, key, newValue) {
      // console.log(`set ${key}: ${newValue}`)
      // 设置属性值
      target[key] = newValue

      trigger(target, key)
    },
  })
}
```

最后再回到前面的例子，你会发现我们成功了!!!

![](https://cdn.wallleap.cn/img/pic/illustration/202308041032640.jpeg)

## 4.# watch 的新值和旧值

到目前为止，我们实现了 `watch` 最基本的功能，感知其数据的变化并执行对应的回调。

接下来我们再实现一个基础功能：**在回调函数中获取新值与旧**值。

```js

watch(state, (newVal, oldVal) => {
  // xxx
})

```

**新值和旧值主要在于获取时机不一样，获取方式确实一模一样的，执行 effectFn 即可**

```js
const watch = (source, cb) => {
  let getter
  let oldValue
  let newValue
  // 处理传回调的方式
  if (typeof source === "function") {
    getter = source
  } else {
    getter = () => bfs(source)
  }

  const effectFn = effect(getter, {
    lazy: true,
    scheduler() {
      // 变化后获取新值
      newValue = effectFn()
      cb(newValue, oldValue)
      // 执行回调后将新值设置为旧值
      oldValue = newValue
    }
  })
  // 第一次执行获取值
  oldValue = effectFn()
}

```

**测试一波**

```js
const state = reactive({
  name: "前端胖头鱼",
  age: 100,
  obj2: {
    name: "胖小鱼",
    age: 10,
  },
})

watch(() => state.name, (newValue, oldValue) => {
  console.log("state.name", { newValue, oldValue })
})

state.name = '111'
```

![](https://cdn.wallleap.cn/img/pic/illustration/202308041032641.jpeg)

## 5.# 支持立即调用时机

最后再实现立即调用时机 `immediate`，`watch` 就大功告成啦！

```js
const watch = (source, cb, options = {}) => {
  let getter
  let oldValue
  let newValue
  // 处理传回调的方式
  if (typeof source === "function") {
    getter = source
  } else {
    getter = () => bfs(source)
  }

  const job = () => {
    // 变化后获取新值
    newValue = effectFn()
    cb(newValue, oldValue)
    // 执行回调后将新值设置为旧值
    oldValue = newValue
  }

  const effectFn = effect(getter, {
    lazy: true,
    scheduler() {
      job()
    }
  })
  // 如果指定了立即执行，便执行第一次
  if (options.immediate) {
    job()
  } else {
    oldValue = effectFn()
  }
}


watch(() => state.name, (newValue, oldValue) => {
  console.log("state.name", { newValue, oldValue });
}, { immediate: true });
```

![](https://cdn.wallleap.cn/img/pic/illustration/202308041032642.jpeg)

通过判断 `immediate` 是否为 true 来决定是否一开始就执行 `cb` 回调，且第一次回调的旧值 `oldValue` 应该为 `undefined`。

## 结尾

最近在阅读**霍春阳**大佬的 **《Vue.js 技术设计与实现》**，本文的内容主要来源于这本书，强烈推荐对 Vue 底层实现感兴趣的同学阅读。
