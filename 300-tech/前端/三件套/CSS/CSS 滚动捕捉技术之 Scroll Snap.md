---
title: CSS 滚动捕捉技术之 Scroll Snap
date: 2024-10-17T02:29:18+08:00
updated: 2024-10-17T02:32:57+08:00
dg-publish: false
---

```css
.container {
  overflow-x: scroll;
  /* 方向 如何吸附 */
  scroll-snap-type: x mandatory;    /* mandatory 强制 */
}

.container::-webkit-scrollbar {
  width: 0;
}

.item {
  scroll-snap-align: start;  /* center end */
  scroll-snap-stop: always;  /* 吸附停止 */
}
```
