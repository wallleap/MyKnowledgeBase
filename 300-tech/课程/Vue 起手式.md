---
title: Vue 起手式
date: 2023-05-17 10:56
updated: 2023-05-17 11:07
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - Vue 起手式
rating: 1
tags:
  - web
category: web
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
description: 文章描述
source: null
url: //myblog.wallleap.cn/post/1
---

开始学习 Vue 了

## Vue 的历史

- 读音
	- 读作 View，意为 MVC 中的 V
	- MVC 中的 V 是 Vue 的重点，M 和 C 被简化
- 版本
	- 2013，v0.6、v0.7
	- 2014，v0.8~v0.11
	- 2015，v1.0 —— MVVM
	- 2016，v2.0 —— MV*
	- 2019，v2.6
	- 2020，v3

## Vue 作者

- 尤雨溪，Evan You
- 主要作品：Vue、Vue Router、Vuex、@vue-cli
	- @vue-cli 后由蒋豪群（sodatea）主要维护

> 启发：
> - 产生的价值越大，得到的金钱越多
> - 英文要好

## 官方文档

- 英文：[Vue.js (vuejs.org)](https://v2.vuejs.org/)
- 中文：[Vue.js (vuejs.org)](https://v2.cn.vuejs.org/)

> 渐进式 JavaScript 框架
>  The Progressive JavaScript Framework

## 创建 Vue 项目

### 使用 [Vue CLI](https://cli.vuejs.org/zh/) 来创建这个项目

- 在终端中使用命令 `npm install -g @vue/cli` 安装 @vue-cli
- 安装完成之后运行 `vue --version` 查看版本
- 运行 `vue create vue-demo` 创建一个项目，进行一些配置选择
	- Manually select features
	- Babel, CSS Pre-processors（使用空格选择，完全选择完之后回车确认）
- 创建完成之后按照提示进入目录，`npm run serve` 进行预览

### 自己从零开始搭建 Vue 项目

- 使用 webpack 或者 rollup 

## 实现一个点击加一的按钮

**Vue 实例**

- 两个版本
	- 完整版 `vue.js`、`vue.min.js`
		- 从 HTML 得到视图
		- 同时包含编译器（Compiler）和运行时（Runtime）的构建
	- 运行时版本 `vue.runtime.js`
		- 用 JS 构建视图
		- 需要自己写 h 函数
		- h 也可以交给 vue-loader，把 .vue 文件翻译成 h 构建方法，这样做 HTML 就只有一个 `dia#app` ，SEO 不友好
- `main.js` 中 `const vm = new Vue({})` 
- 编译器负责把模板字符串编译成 JS 渲染函数的代码
- 运行时负责创建 Vue 实例的代码，渲染和修补虚拟 DOM 等。基本上去掉了编译器。

**完整版** Vue 的写法：

在 HTML/模板 中用到了模板字符串，需要用到 Compiler（`{{count}}` → `0`、`v-if`、`v-for`  等）

```html
<div id="app">
  {{count}}
  <button @click="add">+1</button>
</div>
<script>
  new Vue({
	el: '#app',
	// template: `<div>{{count}}<button v-on:click="add">+1</button></div>` // 或者用这种
	data: {
	  count: 0
	},
	methods: {
	  add() {
		this.count += 1
	  }
	}
  })
</script>
```

编译器 → 复杂 → 占用代码体积（用户的）

**不完整版** Vue 的写法：

纯运行时，不需要用到编译器，可以使用 vue-loader，节省了近 40% 代码体积

```html
<div id="app"></div>
<script>
  new Vue({
	el: '#app',
	render(h) {  // h → createElement
	  return h('div', [this.count, h('button', {on: {click: this.add}}, '+1')])
	},
	data: {
	  count: 0
	},
	methods: {
	  add() {
		this.count += 1
	  }
	}
  })
</script>
```

在 `render` 函数中 `return` 参数 `h`

`h` 用来创建标签，基本用法 `h(标签, 内容)`

![](https://cdn.wallleap.cn/img/pic/illustration/202305171627877.png)

- 非完整版中用 template/在 HTML 中写，会报错，只有运行时版本，模版编译器不可用，可以预编译（Vue-loader）或使用包含了编译器的构建版本，**无法将 HTML 编译成视图**
- 完整版中用 h 函数，代码体积会变大，因为 vue.js  有编译  HTML 的功能

**Vue 单文件组件**

新建 `MyDemo.vue` 文件

```vue
<template>  <!-- 视图放这 -->
  <div class="red">
    {{ count }}
    <button @click="add">+1</button>
  </div>
</template>
<script>  <!-- 除了视图的其他选项 -->
export default {
  data() {  // vue-loader 下 data 必须是函数
    return {
      count: 0
    }
  },
  // 上面的 template 中内容会自动翻译成 render(h){ return h(...) }
  methods: {
    add() {
      this.count += 1
    }
  }
}
</script>
<style scoped> /* 样式写这 */
.red {
  color: red;
}
</style>
```

`main.js` 中引入并挂载

```js
import Vue from 'vue'
import MyDemo from './MyDemo.vue'
// 必须加 ./；加不加 .vue 都可以，但是加了更明确
// console.log(MyDemo)
new Vue({
  el: '#app',
  render: h => h(MyDemo)
})
```

MyDemo 是一个对象，有 beforeCreate、beforeDestroy 数组，data、render 等方法，有 methods 对象

可以把 `MyDemo.render.toString()` 打印出来就是 render 方法

