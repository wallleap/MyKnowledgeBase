---
title: 巧用 CSS 动画延迟时间实现复杂动画
date: 2024-09-24T10:05:05+08:00
updated: 2024-09-24T10:20:01+08:00
dg-publish: false
---

例如 HTML 结构

```html
<div class="container">
  <div class="box">
    <div class="eye"></div>
    <div class="eye"></div>
  </div>
  <input type="range" min="0" max="1" step="0.1" value="0" title="progress">
</div>
```

通过 JS 设置变量

```js
const inp = document.querySelector('input[type="range"]')
const container = document.querySelector('.container')
function cal() {
  container.style.setProperty('--progress', inp.value)
}
inp.oninput = cal
cal()
```

CSS 控制动画效果

```css
.box {
  position: relative;
  width: 120px;
  height: 120px;
  border-radius: 100vh;
  animation: colorChange 1s var(--progress) linear forwards;
}
.eye {
  position: absolute;
  top: 30px;
  left: 16px;
  width: 30px;
  height: 30px;
  border-radius: 30px;
  background-color: #ffffff;
  animation: eyeChange 1s var(--progress) linear forwards;
}
.eye:last-of-type {
  left: auto;
  right: 16px;
  transform: rotate(90deg);
}
@keyframes colorChange {
  0% {
    background-color: #ff3030;
  }
  100% {
    background-color: #36ff54;
  }
}
@keyframes eyeChange {
  0% {
    clip-path: polygon(0 100%, 100% 0, 100% 100%, 0 100%);
  }
  100% {
    clip-path: polygon(0 0%, 100% 0, 100% 100%, 0 100%);
  }
}
```

