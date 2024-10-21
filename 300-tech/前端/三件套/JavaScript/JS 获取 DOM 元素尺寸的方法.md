---
title: JS 获取 DOM 元素尺寸的方法
date: 2024-09-19T10:34:06+08:00
updated: 2024-09-19T10:39:08+08:00
dg-publish: false
---

- client：边框以内的尺寸，不包含边框，四舍五入的整数
- offset：布局，包含边框、滚动条的整数
- scroll：整个内容，即使溢出了也包含，整数
- getBoundingClientRect()：对象，最小矩形尺寸（旋转了还是横竖的），含小数
