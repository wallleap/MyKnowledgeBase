---
title: Vue 3 组合式 API
date: 2023-08-16 22:06
updated: 2024-03-28176 110:03
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - Vue 3 组合式 API
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

## 为什么使用组合式 API

选项式 API 业务逻辑过于分散（data、生命周期钩子、computed、watch）

同一业务逻辑的代码应当放在一起

组合式 API（props、setup 函数）

```js
import { ref, watch, computed, onMountd } from 'vue'

const getData = new Promise((resove, reject) => {
  setTimeout(() => {
    resolve(Math.floor(Math.random() * 100))
  }, Math.floor(Math.random() * 1000))
})

export default {
  props: ['msg'],
  setup(props) {
    let count = ref(0) // 响应式数据 data() { return { count: 0 }} 后面需要 return { count }
    const onAddCount = () => count.value++ // methods: { onAddCount() {...} } 注意使用 ref 要用 .value 取值
    const getCount = async () => count.value = await getData() // methods: { async getCount(){...} }
    watch(count, (value, odlValue) => {
      console.log(`count from ${oldValue} to ${value}.`)
    }) // watch: {count(value, oldValue){...}}
    onMounted(getCount) // mounted() { this.getCount() }

    let name = 'TOM'
    let helloText = computed(() => props.msg + ' ' + name.value)  // computed: { helloText() { return this.msg + '' + this.name }}

    return { // template 中要用到的
      count,
      helloText,
      onAddCount
    }
  }
}
```

- 所有逻辑**放到 setup 函数**，第一个参数是 props 对象
- 通过 ref、reactive、toRef 来创建响应式数据
- 视图要用的变量/函数 return 对象
- watch、computed 是个函数
- 生命周期钩子写法由 xxx 变成 onXxx（例如 `mounted` 变成了 `onMounted`），并且 created 和 beforeCreate 不再需要显式写了（没有 `onCreated` 和 `onBeforeCreate`）

```vue
<template>
  <h1>{{helloText.}}</h1>
  <p>The count is {{count}}</p>
  <button @click="onAddCount">count++</button>
</template>
```

App.vue 中

```vue
<template>
  <Child :msg="value" />
  <input type="text" v-model="value" />
</template>

<script>
import {ref} from 'vue'
import Child from './components/child'
export default {
  components: { Child },
  setup() {
    let value = 'hello'
    return {
      value
    }
  }
}
</script>
```

## 生命周期钩子函数

| 选项式 API           | Hook inside setup   |
| ----------------- | ------------------- |
| `beforeCreate`    | 不需要                 |
| `created`         | 不需要                 |
| `beforeMount`     | `onBeforeMount`     |
| `mounted`         | `onMounted`         |
| `beforeUpdate`    | `onBeforeUpdate`    |
| `updated`         | `onUpdated`         |
| `beforeUnmount`   | `onBeforeUnmount`   |
| `unmounted`       | `onUnmounted`       |
| `errorCaptured`   | `onErrorCaptured`   |
| `renderTracked`   | `onRenderTracked`   |
| `renderTriggered` | `onRenderTriggered` |
| `activated`       | `onActivated`       |
| `deactivated`     | `onDeactivated`     |

因为 `setup` 是围绕 `beforeCreate` 和 `created` 生命周期钩子运行的

所以不需要显式地定义他们

在这两个钩子中编写的任何代码都应该直接在 `setup` 函数中编写

## 创建响应式数据

### ref

接受一个内部值并返回一个响应式且可变的 ref 对象

ref 对象具有指向内部值的单个 property.value

```js
const count = ref(0) // count.value
console.log(count.value) // 0 基础类型 → 对象

count.value++
console.log(count.value) // 1
```

### reactive

处理对象，变成响应式数据

```js
const state = reactive({
  foo: 1,
  bar: 2
}) // state.foo state.bar

const fooRef = toRef(state, 'foo')

fooReff.value++
console.log(state.foo) // 2

state.foo++
console.log(fooRef.value) // 3
```

### toRefs

将响应式对象转换为普通对象，其中结果对象的每个 property 都是指向原始对象相应 property 的 ref

```js
const state = reactive({
  foo: 1,
  bar: 2
})

const stateAsRefs = toRefs(state)

// ref 和原始 property 已经“链接”起来了
state.foo++
console.log(stateAsRefs.foo.value) // 2

stateAsRefs.foo.value++
console.log(state.foo) // 3
```

### toRef

可以用来为源响应式对象上的某个 property 新创建一个 ref

然后，ref 可以被传递，它会保持对其源 propertty 的响应式链接

```js
const state = reactive({
  foo: 1,
  bar: 2
})

const toRef = toRef(state, 'foo')

fooRef.value++
console.log(state.foo) // 2

state.foo++
console.log(fooRef.value) // 3
```

## setup

### props 参数

```js
export default {
  props: {
    title: String
  },
  setup(props) {
    console.log(props.title)
  }
}
```

不想写 `props.`

```js
import { toRefs } from 'vue'

setup(props) {
  const { title } = toRefs(props)
  console.log(title.value)
}
```

只要一个

```js
import { toRef } from 'vue'
setup(props) {
  const title = toRef(props, 'title')
  console.log(title.value)
}
```

1. props 是响应式的
2. 直接结构会让 props 丧失响应特性
3. 可以使用 toRefs 来让解构后的对象保持响应特性
4. toRef 可以直接拿到 props 的属性保持响应特性的同时还能配置可选属性

### context 参数

```js
export default {
  setup(props, context) {
    // Attribute（非响应式对象）
    console.log(context.attrs)
    // 插槽（非响应式对象）
    console.log(context.slots)
    // 触发事件（方法）
    console.log(context.emit)
  }
}
```

## setup 语法糖

```vue
<script setup>
import { ref, reactive, computed, watch, watchEffect } from 'vue'

const refState = ref(0)
const name = ref('Tom')
const list = reactive([1, 2, 3, 4, 5])
const obj = reactive({
  name: 'Tom',
  age: 18
})

console.log(refState.value, list) // 在 script 中 ref 需要 .value

const newList = computed(() => {
  return list.filter(item => item > 2)
})

watch(obj, (newVal, oldVal) => {
  console.log(newVal, oldVal)
}, {
  inmmediate: true,
  deep: true
})
// 精确监听
watch(
  () => obj.name,
  () => {
    console.log('name changed')
  }
)
// 监听多个值
watch([refState, name], ([newState, newName], [oldState, oldName]) => {
  // console.log()
})
</script>
<template>
  <!-- ref 在这里不需要 .value -->
  <div>{{refState}}, {{list}}, {{newList}}</div>
</template>
```

- `computed` 中不应该有副作用（除了计算的其他任何操作，异步请求、操作 DOM 等），避免直接修改 `computed` 的值（默认是只读的）
- `computed` 会进行计算，当原数据改变时会触发重新计算
- `watch` 是监听/侦听某个数据的变化，还有两个参数
	- `immediate`（立即执行）
	- `deep`（深度监听）
