---
title: 使用  Vite 搭建官网
date: 2023-08-17T01:54:00+08:00
updated: 2024-08-21T10:32:43+08:00
dg-publish: false
aliases:
  - 使用  Vite 搭建官网
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

## 创建 Vue 3 项目

可以直接使用如下命令创建项目

```sh
npm create vue@latest
```

或者 Vite 官网的方法

```sh
# npm 6.x
npm create vite@latest my-vue-app --template vue

# npm 7+, extra double-dash is needed:
npm create vite@latest my-vue-app -- --template vue

# yarn
yarn create vite my-vue-app --template vue

# pnpm
pnpm create vite my-vue-app --template vue
```

查看 [create-vite](https://github.com/vitejs/vite/tree/main/packages/create-vite) 以获取每个模板的更多细节：`vanilla`，`vanilla-ts`, `vue`, `vue-ts`，`react`，`react-ts`，`react-swc`，`react-swc-ts`，`preact`，`preact-ts`，`lit`，`lit-ts`，`svelte`，`svelte-ts`，`solid`，`solid-ts`，`qwik`，`qwik-ts`。

使用如下命令创建项目，项目名为 `any-ui`，模板为 `vue-ts`

```sh
pnpm create vite any-ui --template vue-ts
```

接着按提示运行

```sh
cd any-ui
pnpm install
pnpm run dev
```

之后生成的目录结构为

```
any-ui
|--node_modules
|-- src
    |-- assets → 资源目录
        |-- vue.svg
    |-- components → 组件目录
        |--HelloWorld.vue
    |-- App.vue
    |-- main.ts
    |-- style.css
    |-- vite-env.d.ts
|-- public
    |--vite.svg
|-- .gitignore
|-- index.html
|-- package.json
|-- pnpm-lock.yaml
|-- README.md
|-- tsconfig.json
|-- tsconfig.node.json
|-- vite.config.ts
```

项目中 `index.html` 是项目模板

主要的有两条：`<div id="app"></div>` 是一个挂载的容器，`<script type="module" src="/src/main.ts"></script>` 类型是 ES Module，引入了入口文件 `main.ts`

`src/main.ts` 中

1. 引入了 vue 中的 createApp 用于创建 Vue App 实例
2. 引入了 App.vue 单文件组件
3. 引入了全局样式文件 `style.css`
4. createApp 接受了 App 组件并将其挂载到了容器上

```js
import { createApp } from 'vue'
import './style.css'
import App from './App.vue'

createApp(App).mount('#app')
```

看下 `src/App.vue` 单文件组件

- 三部分（template、script、style）
- 引入了 HelloWorld 组件并在选项的 components 中写上 HelloWorld
- Vue 3 的 `template` 中支持了根标签不是一个的情况

```js
import HelloWorld from './components/HelloWorld.vue'
export default {
  // 还是可以使用 Vue 2 中的语法
  components: {
    HelloWorld // HelloWorld: HelloWorld 缩写 左边是标签名 右边是组件引用
  },
  // 除了 props、components 放这里面
  setup() {
    // 其他内容
  }
}
```

Vue 2 和 Vue 3 的区别：

- 90% 的写法完全一致
- Vue 3 的 template 支持多个根标签，Vue 2 不支持
- Vue 3 有 `createApp()`，而 Vue 2 的是 `new Vue()`
- `createApp(组件)` 传组件、`new Vue({template,render})` 传选项

还可以是直接使用 setup 语法糖

```ts
<script setup lang="ts">
import HelloWorld from './components/HelloWorld.vue'
</script>
```

在这种写法下 props 使用 `defineProps<{ msg: string }>()`

> 养成良好的 commit 习惯，写好 commit message

重命名 `src` 目录为 `docs`

修改 `index.html` 中的 title 和引入

```html
<!-- ... -->
<title>Any-ui</title>
<!-- ... -->
<script type="module" src="/docs/main.ts"></script>
<!-- ... -->
```

## 让项目支持 scss

```sh
pnpm add sass -D
```

## 安装并初始化 vue-router

### 引入 Vue Router 4

Vue Router 4 是和 Vue 3 配套的版本（Vue 2 用 Vue Router 3）

可以使用命令行查看 vue-router 的所有版本号

```sh
pnpm info vue-router versions
```

安装相应版本

```sh
pnpm add vue-router@4.2.4
```

后面使用

1. 新建 `history` 对象
2. 新建 `router` 对象
3. `app.use(router)`
4. 添加 `<router-view>`
5. 添加 `<router-link>`

在入口文件 `src/main.js` 中引入

```js
// 其他 import 
import { createWebHashHistory, createRouter } from 'vue-router'

const history = createWebHashHistory()
const router = createRouter()
```

在引入的时候输入 `his` 提示三个分别是

- `createMemoryHistory` 内存型路由
- `createWebHashHistory` Hash 型路由
- `createWebHistory` History 型路由

我们需要使用 TS 类型检查，让我们在开发的时候发现问题

故重命名 `src/main.js` 为 `src/main.ts`

修改 `index.html` 中引入 `<script type="module" src="/src/main.ts"></script>`

这样当类型/参数有问题的时候就会出现波浪线，鼠标悬浮就会有相应的说明；<kbd>Ctrl</kbd> + 鼠标单击可以跳转到这个类型声明（定义）处

发现 `createRouter()` 需要传 history 和 routes 等参数

```js
const router = createRouter({
  history: history,
  routes: [
    { path: '/', component: Home } // 路径 对应的组件（需要引入）
  ]
})
```

在上面引入

```ts
import Home from './views/Home'
```

先自行新建一个简单的版本 `src/views/Home.vue`

```vue
<template>
  <h2>This is Home</h2>
  <p>这个是 Home 页面，我会在这里展示一些内容</p>
</template>
```

发现有波浪线，提示 `找不到模块 xxx.vue`

- 出现原因：TypeScript 只能理解 `.ts` 文件，无法理解 `.vue` 文件
- 解决办法：Google 搜索 `Vue 3 can not find module`；创建 `xxx.d.ts`，告诉 TS 如何理解 `.vue` 文件

新建 `src/shims-vue.d.ts`

```ts
declare module '*.vue' {
  import { ComponentOptions } from 'vue'
  const componentOptions: ComponentOptions
  export default componentOptions
}
```

接着修改 `src/main.ts`

```ts
const app = create(App)
app.use(router)
app.mount('#app')
```

让路由组件在 `App.vue` 中展示

```vue
<template>
  <h1>Hello</h1>
  <router-view />
</template>
<script>
export default {
  name: 'App'
}
</script>
```

在路由中新增一个 `main.ts` 中的 `routes`

```ts
const router = createRouter({
  // 其他内容
  routes: [
    { path: '/', component: Home },
    { path: '/doc', component: Doc }
  ]
})
```

再新建一个 About 组件 `src/views/Doc.vue`

```vue
<template>
  <h2>This is Doc</h2>
  <p>这个是 Doc 页面</p>
</template>
```

在 `src/main.ts` 中引入

```ts
// 其他 import
import Doc from './views/Doc'
```

现在访问地址栏 `#/` 后面加上 `doc` 就可以访问到这个路由了

我们需要点击切换

故在 `App.vue` 中修改 `template` 中的内容

```vue
<template>
  <h1>点击下面的跳转相应页面</h1>
  <p><router-link to="/">Home</router-link> | <router-link to="/doc">Doc</router-link></p>
  <br>
  <router-view />
</template>
```

> 每次完成一项就提交代码

## 创建首页和文档页

上面我们已经创建好了 Home 和 Doc 页面，现在完善布局

Home.vue

- Topnav：左边是 logo，右边是 menu
- Banner：文字介绍 + 开始按钮

Doc.vue

- Topnav
- Content：左边是 aside，右边是 main

样式使用 scss，需要安装 sass（如果版本不兼容，可以指定版本）

```sh
pnpm add -D sass
```

修改全局样式文件 `docs/style.css` 为 `docs/style.scss`

添加基本的全局样式

### 封装 Topnav

Home、Doc 页面都有 topnav

重复的就抽离封装

新建 `src/components/Topnav.vue`

```vue
<template>
  <div class="top-nav">
    <div class="logo">Logo</div>
    <ul class="menu">
      <li>菜单1</li>
      <li>菜单2</li>
    </ul>
  </div>
</template>
<script lang="ts">
export default {}
</script>
<style lang="scss" scoped>

</style>
```

### 完善 Home 页面

```vue
<script lang="ts">
import Topnav from '../components/Topnav.vue'

export default {
  components: { Topnav }
}
</script>
<template>
  <Topnav />
  <div class="banner">
    <h1>AnUI</h1>
    <p>一个安友开发的 UI 框架</p>
    <p class="actions">
      <a href="https://github.com/">GitHub</a>
      <router-link to="/doc">开始</router-link>
    </p>
  </div>
</template>
<style lang="scss" scoped>

</style>
```

### 完善 Doc 页面

需要做成响应式页面

先实现手机端页面，再实现 PC 端页面（因为 80% 的用户只用手机上网）

```vue

```

config 配置

库模式：[构建生产版本 | Vite 官方中文文档 (vitejs.dev)](https://cn.vitejs.dev/guide/build.html#library-mode)
