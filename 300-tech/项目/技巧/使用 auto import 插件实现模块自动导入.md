---
title: 使用 auto import 插件实现模块自动导入
date: 2024-10-17T02:19:28+08:00
updated: 2024-10-17T02:25:45+08:00
dg-publish: false
---

安装

```sh
$ npm i -D unplugin-auto-import
```

支持很多脚手架，例如在 vite 中配置 `vite.config.ts`

```js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import AutoImport from 'unplugin-auto-import/vite'

export default defineConfig({
  plugins: [
    vue(),
    AutoImport({
      imports: ['vue', 'vue-router'],  // 自动导入哪些模块
      dirs: ['./src/api'],  // 本地模块，告诉它目录
      dts: './src/auto-import.d.ts',  // TS，自动创建类型文件，把上面用到的模块类型定义都放到这里面
    })
  ]
})
```
