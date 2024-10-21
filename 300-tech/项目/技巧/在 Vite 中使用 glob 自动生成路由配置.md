---
title: 在 Vite 中使用 glob 自动完成路由配置
date: 2024-10-14T01:47:15+08:00
updated: 2024-10-14T01:47:32+08:00
dg-publish: false
---

目录结构能够一定程度反映 routes 配置（约定大于配置）

还差一些 `meta` 信息，可以在每个路径下新建 `page.js`（小程序做法）

```js
export default {
  title: '首页',
  menuOrder: 1
}
```

## 自动生成路由配置

- vue-cli 使用的 webpack，可以用 `require.context`
- vite 可以直接用 `import.meta.glob`

`router/index.js`

```js
import { createRouter, createWebHistory } from 'vue-router'

// 自动生成 routes
const pageModules = import.meta.glob('../views/**/page.js', {
  eager: true,  // 默认是动态导入，设置为 true 就模块导入
  import: 'default'  // 直接导入它的 default
})
const compModules = import.meta.glob('../views/**/index.vue')

const routes = Object.entries(pageModules.map(([pagePath, config]) => {
  let path = pagePath.replace('../views', '').replace('/page.js', '')
  path = path || '/'
  const name = path.split('/').filter(Boolean).join('-') || 'index'
  const compPath = pagePath.replace('page.js', 'index.vue')
  return {
    path,
    name,
    component: compModules[compPath],
    meta: config,
  }
}))

export const router = createRouter({
  history: createWebHistory(),
  routers
})
```

现成的：vite-plugin-pages
