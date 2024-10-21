---
title: 纯 CSS 实现瀑布流布局
date: 2024-09-07T10:37:09+08:00
updated: 2024-09-07T10:37:23+08:00
dg-publish: false
---

```css
/* .container>img*30 */
.container {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-gap: 10px;
  grid-template-rows: masonry;
}

img {
  width: 100%;
}
```
