---
title: Vite 打包目录结构控制
date: 2024-08-27T03:33:59+08:00
updated: 2024-08-27T04:39:47+08:00
dg-publish: false
---

Vite 的配置构建选项中没有找到，接着到 Rollup 中找

[Configuration Options | Rollup (rollupjs.org)](https://rollupjs.org/configuration-options/#output-entryfilenames)

## JS

- 入口文件
- 分包 output.chunkFileNames

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
