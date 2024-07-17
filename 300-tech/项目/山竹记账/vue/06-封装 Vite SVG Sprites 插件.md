---
title: 06-封装 Vite SVG Sprites 插件
date: 2024-05-16 14:38
updated: 2024-05-16 14:39
---

安装两个依赖

```sh
pnpm add svgo svgstore
```

新建 `vite_plugin/svgstore.js`，自己实现一个插件

```js
import path from 'path'  
import fs from 'fs'  
import store from 'svgstore' // 用于制作 SVG Spritesimport { optimize } from 'svgo' // 用于优化 SVG 文件  
  
export const svgstore = (options = {}) => {  
  const inputFolder = options.inputFolder || 'src/assets/icons';  
  return {  
    name: 'svgstore',  
    resolveId(id) {  
      if (id === '@svgstore') {  
        return 'svg_bundle.js'  
      }  
    },  
    load(id) {  
      if (id === 'svg_bundle.js') {  
        const sprites = store(options);  
        const iconsDir = path.resolve(inputFolder);  
        for (const file of fs.readdirSync(iconsDir)) {  
          const filepath = path.join(iconsDir, file);  
          const svgid = path.parse(file).name  
          let code = fs.readFileSync(filepath, { encoding: 'utf-8' });  
          sprites.add(svgid, code)  
        }  
        const { data: code } = optimize(sprites.toString({ inline: options.inline }), {  
          plugins: [  
            'cleanupAttrs', 'removeDoctype', 'removeComments', 'removeTitle', 'removeDesc',  
            'removeEmptyAttrs',  
            { name: "removeAttrs", params: { attrs: "(data-name|data-xxx)" } }  
          ]  
        })  
        return `const div = document.createElement('div')  
          div.innerHTML = \`${code}\`  
          const svg = div.getElementsByTagName('svg')[0]          if (svg) {            svg.style.position = 'absolute'            svg.style.width = 0            svg.style.height = 0            svg.style.overflow = 'hidden'            svg.setAttribute("aria-hidden", "true")          }          // listen dom ready event          document.addEventListener('DOMContentLoaded', () => {            if (document.body.firstChild) {              document.body.insertBefore(div, document.body.firstChild)            } else {              document.body.appendChild(div)            }          })`      }  
    }  
  }  
}
```

`vite.config.ts` 中引入并使用这个插件

```ts 
// @ts-ignore  
import { svgstore } from './src/vite_plugins/svgstore'  
  
export default defineConfig({  
  plugins: [ 
    svgstore(),  
  ],  
})
```

`main.ts` 中引入

```ts
import '@svgstore'
```

在使用的时候 svg>use

```tsx
<svg>
  <use xlinkHref='#cloud'></use>
</svg>
```

Git 命令

```sh
git diff # 展示当前未提交的变动
```

