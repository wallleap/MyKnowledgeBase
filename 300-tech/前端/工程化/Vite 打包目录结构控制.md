---
title: Vite 打包目录结构控制
date: 2024-08-27T03:33:59+08:00
updated: 2024-11-25T10:45:42+08:00
dg-publish: false
---

Vite 打包命令 `vite build`

可能打包会把工具库、框架、自己代码都打包到一起，但根据修改频率，工具库和框架一般变动不大，可以进行单独打包（打包到一起或分包），方便缓存

动态导入会自动分包（路由懒加载的时候）例如 `import('lodash')`

Vite 使用到两个

- 开发 ESBuild
- 打包 Rollup

Vite 的配置构建选项中没有找到，接着到 Rollup 中找

[Configuration Options | Rollup (rollupjs.org)](https://rollupjs.org/configuration-options/#output-entryfilenames)，配置 `manualChunks`

可以是对象，也可以是个函数

对象的时候，属性就是打包之后的名字，例如

```js
export default defineConfig({
  plugins: [vue()],
  build: {
    rollupOptions: {
      manualChunks: {
        a: ['lodash', 'vue'],
        b: ['dayjs']
      },
    },
  },
})
```

如果有很多需要打包到一起的，就可以用函数，在打包的时候会自动调用这个函数，return 的是打包后的文件名

```js
export default defineConfig({
  plugins: [vue()],
  build: {
    rollupOptions: {
      manualChunks(id) {
        // 把在 node_modules 中的包合并
        if (id.include('node_modules')) {
          return 'vendor'
        }
      }
    },
  },
})
```

## JS

- 入口文件
- 分包 `output.chunkFileNames`

## 其他资源

例如 CSS 等

- output.assetFileNames

```ts
import UnoCSS from 'unocss/vite'
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import viteCompression from 'vite-plugin-compression'

// https://vitejs.dev/config/
export default defineConfig({
  envPrefix: ['V_'],
  preview: {
    host: true,
  },
  plugins: [
    vue(),
    UnoCSS(),
    viteCompression({
      threshold: 300000,
    }),
  ],
  build: {
    outDir: 'dist',
    rollupOptions: {
      output: {
        entryFileNames: 'js/[name]-[hash].js',
        chunkFileNames: 'js/[name]-[hash].js',
        assetFileNames(assetInfo) {
          const imgExts = ['png', 'jpg', 'jpeg', 'gif', 'svg', 'webp']
          if (assetInfo.name?.endsWith('.css'))
            return 'css/[name]-[hash].css'
          if (imgExts.some(ext => assetInfo.name?.endsWith(ext)))
            return 'imgs/[name]-[hash].[ext]'
          return 'assets/[name]-[hash].[ext]'
        },
        manualChunks(id) {
          if (id.includes('markdown')) {
            if (id.includes('markdown-it/'))
              return 'markdown-it'
            else if (id.includes('node_modules'))
              return 'markdown-chunk'
            else return 'markdown'
          }
        },
      },
    },
  },
})
```
