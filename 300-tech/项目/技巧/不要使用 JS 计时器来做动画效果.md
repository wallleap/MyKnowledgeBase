---
title: 不要使用 JS 计时器来做动画效果
date: 2024-09-19T11:16:28+08:00
updated: 2024-09-19T04:34:16+08:00
dg-publish: false
---

由于性能或者其他问题，且计时器也不准确

- 硬件：CPU 寄存器（原子钟最准确）
- 系统：操作系统的计时
- 标准：W3C，嵌套超过 4 层，会最小延迟 4ms
- 事件循环：回调函数必须等待执行栈清空

渲染帧和计时器会对不齐，出现空帧（渲染帧浪费）、跳帧（动画不连续）

使用 `requestAnimationFrame()` RAF

```js
function raf() {
  requestAnimationFrame(function() {
    // 动画
    raf()  // 继续下一帧
  })
}
```

永远跟着渲染帧走
