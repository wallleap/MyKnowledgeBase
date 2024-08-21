---
title: 在 Vue 3 中使用一款新的图标库 - IconPark 浅谈
date: 2023-08-04T11:50:00+08:00
updated: 2024-08-21T10:33:31+08:00
dg-publish: false
aliases:
  - 在 Vue 3 中使用一款新的图标库 - IconPark 浅谈
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

小知识，大挑战！本文正在参与“[程序员必备小知识](https://juejin.cn/post/7008476801634680869 "https://juejin.cn/post/7008476801634680869")”创作活动

在日常的开发中，或是为了提高开发效率，或是为了换种方案解决问题，我们可能会需要借助一些工具，所谓 " 工欲善其事,必先利其器 "。话不多说，我们切入正题！

## 它是谁

说到图标,很难绕过 iconfont，今天介绍的是由字节开源的图标库 -iconPark

[官网](https://link.juejin.cn/?target=https%3A%2F%2Ficonpark.oceanengine.com%2Fhome "https://iconpark.oceanengine.com/home")

![](https://cdn.wallleap.cn/img/pic/illustration/202308041151195.png)

iconPark 的官方图标库提供了**2437**个图标，能够满足大部分场景下的使用相比于 IconFont 给我最直观的感受就是图标的统一性很强,分类清晰，最重要的是很强大,这套图标库可以用于 Vue/React 端,由于我在工作中使用的是 vue 技术桟，正巧在最近的 Vue3 项目中使用了 IconPark,下面我们就来聊聊。

注：以下例子仅在 Vue3 项目中使用，Vue2.X 用法相似,可阅读官方文档了解

## 怎么用

这里我们使用 npm 包的方式引入,首先我们需要建立一个 Vue3 的项目 (使用 webpack)

### npm 安装

```sh
npm install @icon-park/vue-next
```

### 按需引入

找到项目根目录下的 **babel.config.js**添加如下代码

```js
module.exports = {
  presets: [
    '@vue/cli-plugin-babel/preset'
  ],
  plugins: [
    [
      'import',
      {
        libraryName: '@icon-park/vue-next',
        libraryDirectory: 'es/icons',
        camel2DashComponentName: false
      },
      'icon'      // 多个组件库时需要加上这个属性,不重复即可。
    ]
  ]
}

```

需要注意的是，你如果要按需引入多个组件库,那么就需要为组件库注明 name,否则会报错。

### 渐入佳境

由于在 main.js 中引入大量组件代码可能不太美观，我将这部分抽离出去放在一个 js 中维护

src 目录下新增 plugins 目录,用 ES6 语法引入并注册

![](https://cdn.wallleap.cn/img/pic/illustration/202308041151196.png)

```js
import { Play } from '@icon-park/vue-next'
export function IconPark (app) {
  app.component('Play', Play)
}
```

main.js 里

```js
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'
import store from './store'
import { IconPark } from '@/plugins/iconPark'
const app = createApp(App)
IconPark(app)
app.use(store).use(router).mount('#app')

```

可以看到，上面的 IconPark 函数传入一个参数 app,执行了 app.component,引入的图标以组件的形式被全局注册，于是意味着这些图标会以组件的形式使用。

### 来个图标

```vue
<play theme="filled" size="60" fill="#3a5de7" />
```

渲染到页面上

![](https://cdn.wallleap.cn/img/pic/illustration/202308041151197.png)

当然，颜色，大小，格式等等,官网都提供了灵活的配置,可以自行选择,如果你觉得这 2000 多个图标还不够用，或者说没你想要的，可以给官方给出补充建议。好吧，我们聊到这里。
