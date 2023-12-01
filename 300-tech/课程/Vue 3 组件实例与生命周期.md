---
title: Vue 3 组件实例与生命周期
date: 2023-08-12 20:49
updated: 2023-08-12 20:49
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - Vue 3 组件实例与生命周期
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

## 应用实例与组件实例

`const app = Vue.createApp(App)` 创建 app 应用实例

`const vm = app.mount('#app')` 创建 vm 根组件实例

在 Vue 中使用的 `this` 就是根组件实例 `vm`，可以通过修改 `vm.xxx` 修改 `data` 中的数据

## 生命周期

![](https://cdn.wallleap.cn/img/pic/illustration/202308122059343.png)

Vue 暴露了一些函数供我们使用，称为生命周期钩子函数

如果需要看这些阶段，可以创建一个自定义组件，然后 `console.log` 一些内容，例如 `data` 中的数据（`this.xxx`）、`this.$el`，更新的时候使用选择器展示 `innerText`，销毁的时候把子组件上 `v-if` 后的变量设置为 `false`

### beforeCreate

### created

### beforeMount

### mounted

### beforeUpdate

### updated

### beforeUnmount

### unmounted

### 常见问题

1、有哪些生命周期钩子

- create、mount、update、unmount

2、在那个生命周期钩子里进行数据请求

- 可以在 `created` 中
- 在后面也可以（既然在 `created` 中可以，那就直接在 `created` 中）

3、父子组件嵌套，哪个先 mounted

不确定

4、父子组件嵌套时，如果希望在所有组件视图都渲染完成后再执行操作，该如何做

```js
mounted() {
  this.$nextTick(function() {
    // 仅在渲染整个视图之后运行的代码
  })
}
```
