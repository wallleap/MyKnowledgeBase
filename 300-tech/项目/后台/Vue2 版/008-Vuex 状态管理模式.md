---
title: 008-Vuex 状态管理模式
date: 2023-06-20 18:43
updated: 2023-06-28 18:25
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - 008-Vuex 状态管理模式
rating: 1
tags:
  - 项目
category: web
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
description: 文章描述
source: #
url: //myblog.wallleap.cn/post/1
---

Vuex 是一个专为 Vue.js 应用程序开发的**状态管理模式**。它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。

平常 Vue 的写法：

```js
new Vue({
  // state
  data () {
    return {
      count: 0
    }
  },
  // view
  template: `
    <div>{{ count }}</div>
  `,
  // actions
  methods: {
    increment () {
      this.count++
    }
  }
})
```

上述状态自管理应用包含：

- **state**，驱动应用的数据源（简单理解为数据）；
- **view**，以声明方式将 **state** 映射到**视图**；
- **actions**，响应在 **view** 上的用户输入导致的状态变化

![这种的是“单向数据流”](https://cdn.wallleap.cn/img/pic/illustration/202306202221956.png)

把组件的**共享状态抽取**出来，以一个**全局**单例模式管理。在这种模式下，我们的组件树构成了一个巨大的“视图”，不管在树的哪个位置，任何组件都能获取状态或者触发行为！

通过定义和隔离状态管理中的各种概念并通过强制规则维持视图和状态间的独立性，我们的代码将会变得更结构化且易维护。

![](https://cdn.wallleap.cn/img/pic/illustration/202306202222229.png)

## 安装 Vuex

如果用的脚手架而且选了 Vuex 则可以跳过

安装：

```sh
pnpm add vuex --save
```

v3.x 对应的是 Vue 2 <https://v3.vuex.vuejs.org/zh/>

v4.x 对应的是 Vue 3 <https://vuex.vuejs.org/zh/>

## 了解

每一个 Vuex 应用的核心就是 store（仓库）。“store”基本上就是一个容器，它包含着你的应用中大部分的**状态 (state)**。

创建一个 store，需要提供一个初始 state 对象和一些 mutation

```js
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  }
})
```

可以通过 `store.state` 来获取状态对象，以及通过 `store.commit` 方法触发状态变更

```js
store.commit('increment')

console.log(store.state.count) // -> 1
```

Vuex 提供了一个从根组件向所有子组件，以 `store` 选项的方式“注入”该 store 的机制，在 Vue 组件中访问 `this.$store` property

```js
new Vue({
  el: '#app',
  store， // store: store 简写
})
```

其他地方使用

```js
methods: {
  increment() {
    this.$store.commit('increment')
    console.log(this.$store.state.count)
  }
}
```

在 Vue 2 项目中使用 Vuex 的**一般步骤**如下：

1. 安装 Vuex：使用 npm 或 yarn 安装 Vuex。
2. 创建 store：创建一个 store.js 文件，导入 Vue 和 Vuex，并使用 Vue.use(Vuex) 来注册 Vuex 插件。然后创建一个新的 Vuex.Store 实例，将其导出。
3. 定义状态：在 store.js 文件中定义状态，包括 state、mutations、actions、getters 等。
4. 注册 store：在 Vue 实例中注册 store，将 store 作为一个选项传递给 Vue 实例。
5. 在组件中使用：在组件中使用 Vuex，可以使用计算属性和方法来访问 store 中的状态，可以使用 mutations 和 actions 来改变 store 中的状态。

例如，在 store.js 文件中定义一个状态：

```js
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  },
  actions: {
    incrementAsync ({ commit }) {
      setTimeout(() => {
        commit('increment')
      }，1000)
    }
  },
  getters: {
    doubleCount: state => state.count * 2
  }
})
```

然后，在 Vue 实例中注册 store：

```js
new Vue({
  el: '#app',
  store,
  template: '<App/>',
  components: { App }
})
```

最后，在组件中使用 Vuex：

```js
<template>
  <div>
    <p>Count: {{ count }}</p>
    <p>Double Count: {{ doubleCount }}</p>
    <button @click="increment">Increment</button>
    <button @click="incrementAsync">Increment Async</button>
  </div>
</template>

<script>
export default {
  computed: {
    count () {
      return this.$store.state.count
    },
    doubleCount () {
      return this.$store.getters.doubleCount
    }
  },
  methods: {
    increment () {
      this.$store.commit('increment')
    },
    incrementAsync () {
      this.$store.dispatch('incrementAsync')
    }
  }
}
</script>
```

## 在项目中使用

### 登录获取的 token 存入 Vuex 中

登录成功了返回了 token，是一个令牌，验证用户权限（是否能访问某页面）

多个页面都需要这个 token，所以使用 Vuex

在 `src/store/index.js` 中定义，state 里的 token 变量，以及更新 token 的 `updateToken` mutation 函数

```js
export default new Vuex.Store({
  state: {
    // 1. 用来存储登录成功之后，得到的 token
    token: ''
  },
  mutations: {
    // 2. 更新 token 的 mutation 函数
    updateToken(state，newToken) {
      state.token = newToken
    }
  }
})
```

在 `src/views/login/index.vue` 中，在成功后，调用 vuex 里的 mutations 方法

可以直接调用 / **映射调用**

```js
import { mapMutations } from 'vuex'
export default {
  // ...其他
  methods: {
    // 映射到这个组件中，就可以在这个组件里直接调用了
    ...mapMutations(['updateToken']),
    // 登录按钮->点击事件
    async loginFn () {
      this.$refs.loginRef.validate(async valid => {
        if (!valid) return
        // 1. 发起登录的请求
        const { data: res } = await loginAPI(this.loginForm)
        // 2. 登录失败
        if (res.code !== 0) return this.$message.error(res.message)
        // 3. 登录成功
        this.$message.success(res.message)
        // 4. 保存到vuex中
        this.updateToken(res.token)
      })
    }
  }
}
```

在 Vue 的调试插件中可以看到

在登录接口返回响应成功后，提取后台返回的 token 字符串，调用 mutations 方法，把值暂存到 vuex 的 state 内变量上，但是仅仅在**内存中**，刷新后 state 里 token 变量会变成空字符串 (又相当于没登录一样)

### 持久化存储 Vuex

如果在保存到 vuex 时，它能自动**保存到浏览器本地**，默认从浏览器本地取呢?

自己写 `localStorage.setItem` 等 需要一个个写，很麻烦

这里介绍一个 vuex 的插件包叫做 `vuex-persistedstate@3.2.1` 版本 (配合 vue2 使用，默认最新版是配合 vue3 使用)

1、下载此包到当前工程中

```sh
pnpm add vuex-persistedstate@3.2.1
```

2、在 `src/store/index.js` 中，导入并配置

```js
import Vue from 'vue'
import Vuex from 'vuex'
import createPersistedState from 'vuex-persistedstate'
Vue.use(Vuex)
export default new Vuex.Store({
  state: {
    // 1. 用来存储登录成功之后，得到的 token
    token: ''
  },
  mutations: {
    // 2. 更新 token 的 mutation 函数
    updateToken (state，newToken) {
      state.token = newToken
    }
  },
  // 配置为 vuex 的插件
  plugins: [createPersistedState()]
})
```

除了五大核心概念，还有插件选项，在数组中调用函数相当于注入了这个插件

3、这次再来重新登录，查看浏览器调试工具 vuex 和浏览器本地存储位置，是否自动同步进入

4、刷新网页看调试工具里 vuex 的默认值确实从本地取出了默认值，保证了 vuex 的持久化

## 总结

store 只能是唯一的一个

stae 里定义全局状态变量

mutations 里唯一同步修改 state 里的值

```js
store.commit('要触发的mutations函数名')  // 是在 mutations 中定义的修改函数
```

actions 里定义异步代码，一般是调用后端接口的时候，然后把值 commit 给 mutations

```js
store.dispatch('要触发的actions函数名')  // 在 actions 中定义的函数
```

modules：分模块 + 开启命名空间
