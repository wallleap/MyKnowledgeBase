---
title: 保姆级 Vue 3 开发文档
date: 2023-08-04 17:06
updated: 2023-08-04 17:06
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - 保姆级 Vue 3 开发文档
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

大家好，我是沐华。最近一个粉丝公司项目由 Vue2 升级到 Vue3 了，他一下子不适应，有好多不会用的，所以我就写了这篇开发文档，包含了 Vue3 项目开发中使用的所有语法，希望所有像他一样还不熟的伙伴快速上手 Vue3 项目开发

本文所有语法为 `Vue 3.2.41` 版本，如果是 3.0 到 3.2 以内的版本个别地方可能会有些许不一样，但绝大部分都是一样的

## 开发文档

### 获取 this

Vue2 中每个组件里使用 this 都指向当前组件实例，this 上还包含了全局挂载的东西、路由、状态管理等啥啥都有

而 Vue3 组合式 API 中没有 this，如果想要类似的用法，有两种，一是获取当前组件实例，二是获取全局实例，如下自己可以去打印出来看看

```
<script setup>
import { getCurrentInstance } from 'vue'

// proxy 就是当前组件实例，可以理解为组件级别的 this，没有全局的、路由、状态管理之类的
const { proxy, appContext } = getCurrentInstance()

// 这个 global 就是全局实例
const global = appContext.config.globalProperties
</script>
```

### 全局注册 (属性/方法)

Vue2 中我们要往全局上挂载东西通常就是如下，然后在所有组件里都可以通过 `this.xxx` 获取到了

```
Vue.prototype.xxx = xxx
```

而 Vue3 中不能这么写了，换成了一个能被所有组件访问到的全局对象，就是上面说的全局实例的那个对象，比如在 `main.js` 中做全局注册

```
// main.js
import { createApp } from 'vue'
import App from './App.vue'
const app = createApp(App)
// 添加全局属性
app.config.globalProperties.name = '沐华'
```

在其他组件中调用

```
<script setup>
import { getCurrentInstance } from 'vue'
const { appContext } = getCurrentInstance()

const global = appContext.config.globalProperties
console.log(global.name) // 沐华
</script>
```

### template

Vue2 中只能有一个根节点，而 Vue3 中支持多个根节点，这个大家都知道

其实本质上 Vue3 每个组件还是一个根节点，因为 **DOM 树只能是树状结构的**，只是 Vue3 在编译阶段新增了判断，如果当前组件不只一个根元素，就添加一个 `fragment` 组件把这个多根组件的给包起来，相当于这个组件还是只有一个根节点。而 `fragment` 跟 `keep-alive` 一样是一个不会被渲染出来的内置组件

```
<template>
    <div>1</div>
    <div>2</div>
</template>
```

### 获取 DOM

Vue3 中获取 DOM 如下

```
<template>
    <el-form ref="formRef"></el-form>
    <child-component />
</template>
<script setup lang="ts">
import ChildComponent from './child.vue'
import { getCurrentInstance } from 'vue'
import { ElForm } from 'element-plus'

// 方法一，这个变量名和 DOM 上的 ref 属性必须同名，会自动形成绑定
const formRef = ref(null)
console.log(formRef.value) // 这就获取到 DOM 了

// 方法二
const { proxy } = getCurrentInstance()
proxy.$refs.formRef.validate((valid) => { ... })

// 方法三，比如在 ts 里，可以直接获取到组件类型
// 可以这样获取子组件
const formRef = ref<InstanceType<typeof ChildComponent>>()
// 也可以这样 获取 element ui 的组件类型
const formRef = ref<InstanceType<typeof ElForm>>()
formRef.value?.validate((valid) => { ... })
</script>
```

### 初始化

Vue2 中进入页面就请求接口，或者其他一些初始化的操作，一般放在 `created` 或 `mounted`，而 Vue3 中 `beforeCreated` 和 `created` 这俩钩子就不用了，因为 setup 在这俩之前执行，还要这俩的话就多此一举了

所以以前用在 `beforeCreated / created / beforeMount / mounted` 这几个钩子里的内容，在 Vue3 中可以直接放在 `setup` 里，或者放在 `onMounted/onBeforeMount` 里

```
<script setup>
import { onMounted } from 'vue'

// 请求接口函数
const getData = () => {
    xxxApi.then(() => { ... })
}

onMounted(() => {
    getData()
})
</script>
```

### 解除绑定

Vue2 中一般清除定时器、监听之类的操作有两种方法：

- 一是用 `$once` 搭配 `hook: BeforeDestroy` 使用，这个 Vue3 不支持了
- 二是用 `beforeDestroy / deactivated` 这俩钩子，Vue3 中只是把钩子函数重命名了一下

```
<script setup>
import { onBeforeUnmount, onDeactivated } from 'vue'

// 组件卸载前，对应 Vue2 的 beforeDestroy
onBeforeUnmount(() => {
    clearTimeout(timer)
    window.removeAddEventListener('...')
})

// 退出缓存组件，对应 Vue2 的 deactivated
onDeactivated(() => {
    clearTimeout(timer)
    window.removeAddEventListener('...')
})
</script>
```

### ref 和 reactive

这两都是用于创建响应式对象，`ref` 通常用于创建基础类型，`reactive` 通常用于创建响应式，这是官方推荐的，现实中也不尽然，有人也用 `ref` 来定义数组，也有人一个组件只定义一个 `reactive`，所有数据都放里面，就像 Vue2 的 `data` 一样，也有人都用

需要知道的有两点：

- `ref` 如果传入的是引用类型，内部源码也是调用 `reactive` 来实现的
- `ref` 返回的属性在 `template` 中使用，直接用就是了，但是在 JS 中使用，需要通过 `.value` 获取，如下。因为 ref 返回的是一个包装对象

```
<template>
    <div>{{ count }}</div>
</template>
<script setup>
import { ref, reactive } from 'vue'
const count = ref(1)

// 有人这么用 
const arr = ref([])
console.log(arr.value) // []

// 也有人这么用，一个组件里所有的属性全部定义在一个对象里，有点 Vue2 data 的味道
const data = reactive({
    name: '沐华',
    age: 18,
    ...
})
console.log(data.name) // 沐华

// 也有人一个组件里 ref 和 reactive 两种都用，随便你
</script>
```

为什么 `ref` 要返回一个包装对象？Vue2 中 `data` 都是返回一个对象这都知道

因为对象引用类型，可以用来做代理或劫持，如果只返回基础类型的话，存储在栈中，执行栈里执行完就回收了，没有办法添加代理或劫持，自然就没办法追踪后续的变化，所以不得不返回一个对象，这样才能有响应式

### toRef 和 toRefs

这两共同点就是用来创建响应式的引用的，主要用来取出响应式对象里的属性，或者解构响应式对象，解构出来的属性值依然是响应式属性，如果不用这两直接解构的话是会丢失响应式效果的

主要就是方便我们使用直接变量 `xxx`，而不需要 `data.xxx`。并且我们修改 `xxx` 的时候也是直接修改源对象属性的

这两的区别：带 s 和不带 s，就是单数和复数嘛，意思就是取一个和取一堆咯

```
<script setup>
import { reactive, toRef, toRefs } from 'vue'

const data = reactive({
    name: '沐华',
    age: 18
})

// 这样虽然能拿到 name / age，但是会变成普通变量，没有响应式效果了
const { name, age } = data

// 取出来一个响应式属性
const name = toRef(data, 'name')

// 这样解构出来的所有属性都是有响应式的
const { name, age } = toRefs(data)

// 不管是 toRef 还是 toRefs，这样修改是会把 data 里的 name 改掉的
// 就是会改变源对象属性，这才是响应式该有的样子
name.value = '沐沐华华'
</script>
```

### watch

`watch` 就是用来监听一个已有属性，发生变化的时候做某些操作，Vue2 中常用有如下三种写法

```
watch: {
    userId: 'getData',
    userName (newName, oldName) {
        this.getData()
    },
    userInfo: {
        handler (newVal, newVal) { this.getData() },
        immediate: true,
        deep: true
    }
}
```

而 Vue3 中监听的写法就丰富得多了

Vue3 的 `watch` 是一个函数，能接收三个参数，参数一是监听的属性，参数二是接收新值和老值的回调函数，参数三是配置项

```
<script setup>
import { watch, ref, reactive } from 'vue'

const name = ref('沐华')
const data = reactive({
    age: 18,
    money: 100000000000000000000,
    children: []
})

// 监听 ref 属性
watch(name, (newName, oldName) => { ... })

// 监听其他属性、路由或者状态管理的都这样
watch(
    () => data.age, 
    (newAge, oldAge) => { ... }
)

// 监听多个属性，数组放多个值，返回的新值和老值也是数组形式
watch([data.age, data.money], ([newAge, newMoney], [oldAge, oldMoney]) => { ... })

// 第三个参数是一个对象，为可配置项，有5个可配置属性
watch(data.children, (newList, oldList) => { ... }, {
    // 这两个和 Vue2 一样，没啥说的
    immediate: true,
    deep: true,
    // 回调函数的执行时机，默认在组件更新之前调用。更新后调用改成post
    flush: 'pre', // 默认值是 pre，可改成 post 或 sync
    // 下面两个是调试用的
    onTrack (e) { debugger }
    onTrigger (e) { debugger }
})
</script>
```

关于副作用，`watch` 和后面说到的 `watchEffect` 是一样的。

在 `watch` 回调函数中能接收第三个参数 `onInvalidate`，为清除副作用的函数，首次触发监听的回调函数 (handler) 不会执行 `onInvalidate`，之后每次触发**默认**会先执行 `onInvalidate`

就是说**默认**它的执行机制在更新之前调用，比如如下代码，当 `key` 触发更新时会先打印 `222` 再打印 `沐华`，如果需要在更新之后调用，可以在 `watch` 第三个配置项中添加 `flush: post`，

```
// 回调函数接收一个参数，为清除副作用的函数
watch(key, (newKey, oldKey, onInvalidate) => {
    console.log('沐华')
    // 获取DOM默认获取到的是更新前的dom，如果是flush: post，可以获取到更新后的dom
    console.log('DOM节点：', dom.innterHTML)
    onInvalidate(() => {
        console.log(2222)
    })
})
```

`onInvalidate` 的使用场景就是：比如监听的回调函数 (handler) 里有一些异步操作，当再次触发 `watch` 的时候可以用它来对前面未完成的异步任务执行取消/忽略/重置/初始化某些操作，比如取消上一次触发 `watch` 时未完成的请求

监听还没完呢

![](https://cdn.wallleap.cn/img/pic/illustration/202308041706882.jpeg)

### watchEffect

Vue3 中除了 `watch` 还增加了一个 `watchEffect`。区别是：

- `watch` 是对传入的一个或多个值进行监听，触发时会返回新值和老值，且默认第一次不会执行
- `watchEffect` 是传入一个立即执行函数，所以默认第一次就会执行，且不需要传入监听内容，会自动收集函数内的数据源作为依赖，当依赖发生变化时会重新执行函数 (有点像 `computed` 的味道)，而且不会返回新值和老值
- 清除副作用和副作用的刷新时机是一样的，区别是 `watch` 中会作为回调的第三个参数传入，`watchEffect` 中是回调函数的第一个参数
- 正常情况下组件销毁/卸载后这两都会自动停止监听，但也有例外，比如异步的方式，在 `setTimeout` 里创建的监听就都需要手动停止监听，停止方式如下

```js
// 监听方法赋值
const unwatch = watch('key', callback)
const unwatchEffect = watchEffect(() => {})
// 需要停止监听的时候，手动调用停止监听
unwatch()
unwatchEffect()
```

`watchEffect` 使用：

```vue
<script setup>
import { watchEffect } from 'vue'

// 正常使用
watchEffect(() => {
    // 会自动收集这个函数使用到的属性作为依赖，进行监听
    // 监听的是 userInfo.name 属性，不会监听 userInfo
    console.log(userInfo.name)
})

// 有两个参数，参数一是触发监听回调函数，参数二是可选配置项
watchEffect(() => {...}, {
    // 这里是可配置项，意思和 watch 是一样的，不过这只有3个可配置的
    flush: 'pre',
    onTrack (e) { debugger }
    onTrigger (e) { debugger }
})

// 回调函数接收一个参数，为清除副作用的函数，和 watch 的同理
watchEffect(onInvalidate => {
    console.log('沐华')
    onInvalidate(() => {
        console.log(2222)
    })
})
</script>
```

`watchEffect` 如果需要修改配置项 `flush` 为 `post` 或 `sync` 时，可以直接使用别名，如下

```js
watchEffect(() => {...}, {
    flush: 'post',
})
// 和下面这个是一样的
watchPostEffect(() => {})
-----------------------------
watchEffect(() => {...}, {
    flush: 'sync',
})
// 和下面这个是一样的
watchSyncEffect(() => {})
```

### computed

Vue2 中 `computed` 最见的使用场景一般有： `mapGetters/mapState` 获取状态管理的属性、 获取 url 上的属性、条件判断、类型转换之类的，支持函数和对象两种写法

而 Vue3 中 `computed` 不再是一个对象，而是一个函数，用法其实基本差不多，函数第一个参数是侦听器源，用于返回计算的新值，也支持对象写法，第二个参数可用于调试

```js
<script setup>
import { computed } from 'vue'
const props = defineProps(['visible', 'type'])
const emit = defineEmits(["myClick"])

// 函数写法，计算类型
const isFirst = computed(() => props.type === 1)

// 对象写法
const status = computed({
    get () { return props.visible }, // 相当于 Vue2中的 this.visible
    set (val) { emit('myClick', val) } // 相当于 Vue2中的 this.$emit('input', val)
})

// computed 第二个参数也是一个对象，调试用的
const hehe = computed(参数一上面两种都可， {
    onTrack (e) { debugger }
    onTrigger (e) { debugger }
})
</script>
```

### nextTick

`nextTick` 的使用方法，除了不能用 `this` 其他的和 Vue2 一模一样，还是三种方式

```js
<script setup>
import { nextTick} from 'vue'

// 方式 一
const handleClick = async () => {
  await nextTick()
  console.log('沐华')
}

// 方式二
nextTick(() => {
    console.log('沐华')
})

// 方式三
nextTick().then(() => {
    console.log('沐华')
  })
</script>
```

### mixins 和 hooks

Vue2 中逻辑的抽离复用一般用 `mixins`，缺点有三：

- 没有独立命名空间，mixins 会和组件内部产生命名冲突
- 不去翻代码不知道引入的 mixins 里面有啥
- 引入多个 mixins 时不知道自己使用的是来自哪一个 mixins 的

Vue3 中逻辑抽离复用的 `hooks` 语法，其实就是一个函数，可以传参，拿返回值来用。或者可以这样理解：平时要封装公用的方法是怎么写的？Vue3 里就可以怎么写

```js
// xxx.js
expport const getData = () => {}
export default function unInstance () {
    ...
    return {...}
}

// xxx.vue
import unInstance, { getData } from 'xx.js'
const { ... } = unInstance()
onMounted(() => {
    getData()
})
```

关于 `hooks` 如何写出更优雅的代码，还个需要多写，多实践，这不是几句话几行代码就能熟练运用的

### 组件通信

Vue3 组件通信的方式，有如下几种

- props + defineProps
- defineEmits
- defineExpose / ref
- useAttrs
- v-model(支持多个)
- provide / inject
- Vuex / Pinia

关于 Vue 组件通信的使用方式，我以前写过一篇文章，用法上面都介绍的很详细了，就不搬过来了

[Vue3的8种和Vue2的12种组件通信](https://juejin.cn/post/6999687348120190983#heading-7 "https://juejin.cn/post/6999687348120190983#heading-7")

### 多个 v-model

Vue2 中每个组件上只能写一个 `v-model`，子组件没有写 `model` 的话，默认在 `props` 接收 `value` 即可，修改就是用 `this.$emit('input')` 事件

Vue3 中每个组件每个组件上支持写多个 `v-model`，没有了 `.sync` 和 `model` 重命名的操作，也不需要了，写 v-model 的时候就需要把命名一起写上去了，如下：

```js
// 父组件写法
<template>
    <child v-model:name="name" v-model:age="age" />
</template>
<script setup>
import { ref } from "vue"
const name = ref('沐华')
const age = ref(18)
</script>

// 子组件
<script setup>
const emit = defineEmits(['update:name', 'update:age'])

const handleClick = () => {
    console.log('点击了')
    emit('update:name', '这是新的名字')
}
</script>
```

### 状态管理

Vuex 用法和 Vue2 基本一样，从 0 开始的话建议直接用 `Pinia` 吧，Pinia 详细使用方式，我以前也写过一篇文章，也不搬过来了

[上手 Vue 新的状态管理 Pinia，一篇文章就够了](https://juejin.cn/post/7075491793642455077 "https://juejin.cn/post/7075491793642455077")

Vuex 4 用法如下

```js
// main.js
import { createApp } from 'vue'
import App from './App.vue'
import Store from './store'
const app = createApp(App)
app.use(Store)
...

// 模块：store/user/index.js
export default {
    state: {},
    getters: {},
    actions: {},
    mutations: {}
}

// store/index.js
import { createStore } from 'vuex'
import user from './user'
const store = createStore({
    state: {},
    getters: {},
    actions: {},
    mutations: {},
    modules: {
        user
    }
})
export default store

// 需要用到状态管理的 .vue 文件里
<script setup>
import { useStore } from 'vuex' 
const store = useStore()
// 打这个 store 打印出来看下，相当于 Vue2 里的 this.$store，用法基本一样
console.log(store)
</script>
```

### 路由

Vue-Router 4 使用如下，主要是 `route` 和 `router` 打印出来看一下就知道了

```js
// main.js
import { createApp } from 'vue'
import App from './App.vue'
import Router from './router'
const app = createApp(App)
app.use(Router)
...

// router/index.js
import { createRouter, createWebHistory, RouteRecordRaw } from 'vue-router'
// 这个 routes 数组里面就和 vue2 一样写
const routes = [  // ts版这行就是 const routes: Array<RouteRecordRaw> = [
  { path: '/', redirect: { name: 'login' } }
]
const router = createRouter({
  history: createWebHistory(process.env.BASE_URL), // 项目没用这个就不传
  routes
})
export default router

// 需要用到路由的 .vue 文件里
<script setup>
import { useRoute, useRouter } from "vue-router"
// route 对应 Vue2 里的 this.$route
const route = useRoute()
// router 对应 Vue2 里的 this.$router
const router = useRouter()
</script>
```

### CSS 样式穿透

Vue2 中在 `scoped` 中修改子组件或者组件库中的组件样式，改不了的情况下，就可以用样式穿透，不管是 `Less` 还是 `SASS` 都是用 `/deep/ .class {}` 来做样式穿透，而 Vue3 中就不支持 `/deep/` 的写法了，换成 `:deep(.class)`

```vue
<style lang="scss" scoped>
// 这样写不生效的话
.el-form {
    .el-form-item { ... }
}
// Vue2 中这样写
/deep/ .el-form {
    .el-form-item { ... }
}
// Vue3 中这样写
:deep(.el-form) {
    .el-form-item { ... }
}
</style>

// 别再一看没生效，就这样加一个没有 scoped 的 style 标签了，一鼓脑全都加到全局上去了
// 除非是全局都要改的
// 还有些需要加到全局的场景，但只改当前页的，比如有些ui组件是挂在全局上的，可以加个当前页独有的类名就是了
// <style lang="scss">
//  .el-form {
//     .el-form-item { ... }
//  }
// </style>
```

### CSS 绑定 JS 变量

就是 CSS 中可以使用 JS 的变量来赋值，如下

```vue
<template>
    <div class="name">沐华</div>
</template>
<script setup>
import { ref } from "vue"
const str = ref('#f00') // 红色
</script>
<style scoped lang="scss">
.name {
    background-color: v-bind(str); // JS 中的色值变量 #f00 就赋值到这来了
}
</style>
```

## 结语

如果本文对你有一点点帮助，点个赞支持一下吧，你的每一个【赞】都是我创作的最大动力 ^\_^

更多前端文章，或者加入前端交流群，欢迎关注公众号【沐华说技术】，大家一起共同交流和进步呀
