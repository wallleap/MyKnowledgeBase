---
title: 000-如何在 Vue 2 项目中使用 Vite
date: 2023-07-10T02:22:00+08:00
updated: 2024-08-21T10:33:31+08:00
dg-publish: false
aliases:
  - 000-如何在 Vue 2 项目中使用 Vite
author: Luwang
category: 分类
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
  - 项目
url: //myblog.wallleap.cn/post/1
---

[Vite | 下一代的前端工具链 (vitejs.dev)](https://cn.vitejs.dev/)

到 Vite 文档中找插件

## 创建项目

使用 Vite 创建 VueJS 项目，默认是 Vue 3（`my-vue-app` 改成自己的项目名）

```sh
pnpm create vite my-vue-app --template vue
cd my-vue-app
pnpm install
```

## 安装依赖

接着安装 [vite-plugin-vue2](https://github.com/underfin/vite-plugin-vue2)，让 Vue2 可以使用 Vite

请严格按照以下给出选择安装对应插件

- Vue 3 单文件组件：[@vitejs/plugin-vue](https://www.npmjs.com/package/@vitejs/plugin-vue)
- Vue 3 JSX：[@vitejs/plugin-vue-jsx](https://github.com/vitejs/vite-plugin-vue/tree/main/packages/plugin-vue-jsx)
- Vue 2.7 单文件组件：[@vitejs/plugin-vue2](https://github.com/vitejs/vite-plugin-vue2)
- Vue 2.7 JSX：[@vitejs/plugin-vue2-jsx](https://github.com/vitejs/vite-plugin-vue2-jsx)
- Vue 2.6 及以下：[vite-plugin-vue2](https://github.com/underfin/vite-plugin-vue2)

```sh
# 选择一个安装即可，我需要的是 Vue 2.7 故安装
pnpm add @vitejs/plugin-vue2 -D
```

安装 Vue 2 替换掉 Vue 3

```sh
pnpm add vue@2
```

替换 vue-router@4

```sh
pnpm add vue-router@3
```

安装 vuex@3

```sh
pnpm add vuex@3
```

安装 vue-template-compiler

```sh
npm i vue-template-compiler
```

## 安装 CSS 预处理器

```sh
# .scss and .sass
pnpm add -D sass
# .less
pnpm add -D less
# .styl and .stylus
pnpm add -D stylus
```

## 修改 vite.config.js 文件

```js
import path from 'node:path'
import { defineConfig } from 'vite'
// import vue from '@vitejs/plugin-vue' // Vue 3  
import vue from '@vitejs/plugin-vue2' // Vue 2.7  
// import { createVuePlugin } from 'vite-plugin-vue2' // Vue <= 2.6

const isProduction = process.env.NODE_ENV === 'production'

// https://vitejs.dev/config/
export default defineConfig({
  base: isProduction ? './' : '/',
  plugins: [
    vue(),
  ],
  resolve: {
    alias: [
      {
        find: '@',
        replacement: path.resolve(__dirname, 'src'),
      },
    ],
  },
  server: {
    host: '0.0.0.0',
    port: 8000,
  },
})
```

需要注意在 Vite 项目中使用的变量是 `import.meta.env`

vite.config.js 中仍使用 `process.env.NODE_ENV`

在 index.html 中使用 env 变量或其他变量，在社区插件中找到了插件：

- [vite-plugin-html](https://github.com/anncwb/vite-plugin-html) - Plugin to minimize and use ejs template syntax in `index.html`.
