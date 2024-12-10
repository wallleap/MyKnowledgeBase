---
title: 统一 Vite 中的图片转换逻辑
date: 2024-11-25T10:47:19+08:00
updated: 2024-11-25T10:48:38+08:00
dg-publish: false
---

默认是打包的时候会把小图片（小于 4kb）转成 BASE64

这样生产环境和开发环境图片路径会不一样，想保持一致可以

- 取消这个优化
- 开发环境也同样转成 BASE64

```js
export default defineConfig({
  plugins: [vue()],
  build: {
    assetsInlineLimit: 0  // 所有的都是文件，不转成 base64
  },
})
```

自己写 Vite 插件

```js
import fs from 'node:fs'
const convertDataurl = (limit = 4096) => {
  return {
    name: 'convert-dataurl',
    async transform(code, id) {
      if (process.env.NODE_ENV !== 'development') {
        return
      }
      const imageExtensionRegex = /\.(jpg|jpeg|png|gif|bmp|svg)$/i
      if (!imageExtensionRegex.test(id)) {
        return
      }
      const extension = filename.slice((Math.max(0, filename.lastIndexOf(".")) || Infinity) + 1).toLowerCase()
      const stat = await fs.promises.stat(id)
      if (stat.size > limit) {
        return
      }
      const buffer = await fs.promises.readFile(id)
      const base64 = buffer.toString('base64')
      const dataUrl = `data:image/${extension};base64,${base64}`
      return {
        code: `export default "${dataUrl}"`
      }
    }
  }
}

export default defineConfig({
  plugins: [vue(), convertDataurl()],
  build: {
    assetsInlineLimit: 4096  // 默认 4kb
  },
})
```
