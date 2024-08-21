---
title: 图片取色 UI
date: 2024-02-22T08:50:00+08:00
updated: 2024-08-21T10:32:45+08:00
dg-publish: false
aliases:
  - 图片取色 UI
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
source: 
tags:
  - web
url: //myblog.wallleap.cn/post/1
---

图片 ——> Canvas ——> 获取前三种主要颜色

使用第三方库 Colorthief

[Color Thief (lokeshdhakar.com)](https://lokeshdhakar.com/projects/color-thief/)

```js
import ColorThief from 'colorthief'

const colorThief = new ColorThief()
const imgs = document.querySelectorAll('img')
const figures = document.querySelectorAll('figure')

for (let i = 0; i < imgs.length; i++) {
  const img = imgs[i]
  
  if (img.complete) {
    const colors = colorThief.getPalette(img, 3)
    const [c1, c2, c3] = colors.map((c) => `rgb(${c.join(',')})`)
    figures[i].style.background = `linear-gradient(45deg, ${c1}, ${c2}, ${c3})`
  } else {
    img.addEventListener('load', function() {
      const colors = colorThief.getPalette(img, 3)
      const [c1, c2, c3] = colors.map((c) => `rgb(${c.join(',')})`)
      figures[i].style.background = `linear-gradient(45deg, ${c1}, ${c2}, ${c3})`
    })
  }
}
```

需要等图片加载完再识别

如果需要的不是 rgb 值而是 hex 可以添加下列函数

```js
const rgbToHex = (r, g, b) => '#' + [r, g, b].map(x => {
  const hex = x.toString(16)
  return hex.length === 1 ? '0' + hex : hex
}).join('')

rgbToHex(102, 51, 153); // #663399
```
