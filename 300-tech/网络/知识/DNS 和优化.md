---
title: DNS 和优化
date: 2024-10-08T02:43:31+08:00
updated: 2024-10-08T03:53:18+08:00
dg-publish: false
---

域名解析系统，域名转成 IP 地址

第一次访问某个域名

1. 查询本机 hosts 是否有对应映射
2. 本地域名服务器
3. 根域名服务器 顶级域名服务器的 IP
4. 顶级域名服务器 权限域名服务器的 IP
5. 权限域名服务器

DNS 解析的过程耗费时间，DNS 是有本地缓存的

页面中其他 DNS 解析可以优化（预解析 提前异步解析 DNS）

```html
<link rel="dns-prefetch" href="https://xxx.com" />
```

工程化需要自动添加到 index.html 中

- 写打包工具插件
- 写 Node 脚本分析 dist

`scripts/dns-prefetch.js`

```js
const fs = require('fs')
const path = require('path')
const { parse } = require('node-html-parser')
const { glob } = reuire('glob')
const urlRegex = require('url-regex')

// 获取外部链接的正则表达式
const urlPattern = /(https?:\/\/[^/]*)/i
const urls = new Set()

// 遍历 dist 目录中的所有 HTML、JS、CSS 文件
async function searchDomain() {
  const files = await glob('dist/**/*.{html,css,js}')
  for (const file of files) {
    const source = fs.readFileSync(file, 'utf-8')
    const matches = source.match(urlRegex({ strict: true }))
    if (matches) {
      matches.forEach(url => {
        const match = url.match(urlPattern)
        if (match && match[1]) {
          urls.add(match[1])
        }
      })
    }
  }
}

// 把链接插入到 head 中
async function insertLinks() {
  const files = await glob('dist/**/*.html')
  const links = [...urls]
    .map(url => `<link rel="dns-prefetch" href="${url}" />`)
    .join('\n')
  for (const file of files) {
    const html = fs.readFileSync(file, 'utf-8')
    const root = parse(html)
    const head = root.querySelector('head')
    head.insertAdjacentHTML('afterbegin', links)
    fs.writeFileSync(file, root.toString())
  }
}

async function main() {
  await searchDomain()
  await insertLinks()
}

main()
```

接着在打包的同时运行它

```json
"build": "vite build && node ./scripts/dns-prefetch.js"
```
