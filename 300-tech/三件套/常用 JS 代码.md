---
title: 常用 JS 代码
date: 2023-08-01 14:52
updated: 2024-04-09 12:25
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - 常用 JS 代码
rating: 1
tags:
  - blog
category: 分类
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
description: 文章描述
source: #
url: //myblog.wallleap.cn/post/1
---

## 获取 DOM 节点

```js
document.documentEl // 获取根标签
document.body // 获取 body 标签
document.querySelector(el) // 获取选择器为 el 的元素
document.querySelectorAll(el) // 获取选择器为 el 的元素数组
```

## 操作 DOM 元素

```js
EL.append(el)
/* <p data-test-a='AreYouOK'>test</p> */
document.querySelector('p').dataset['testA']
document.querySelector('p').setAttribute('')
```

```js
document.dataset.theme
```

## CSS 样式

```gist
34086cbddd6e3a2072aadfd109193a02
```

## 暗黑模式

媒体查询匹配暗黑模式，由于亮色模式是默认的，所以可以直接写外面

```css
:root {
  --primary: #333;
}
@media (prefers-color-scheme: light) {
  /* 亮色模式下的样式，默认就是，所以可以不写这里面 */
}
@media (prefers-color-scheme: dark) {
  /* 暗黑模式下的样式 */
  :root {
    --primary: #333;
  }
}
```

JS 监听暗黑模式变化

```js
const darkModeQuery = window.matchMedia('(prefers-color-scheme: dark)')
// 是否是暗黑模式
console.log(darkModeQuery.matches) // true / false
// onchange、addListener / addEventListener（看浏览器支持）
darkModeQuery.addEventListener('change', () => {

})
darkModeQuery.removeEventListener('change', () => {

})
```

总结：

1. CSS 定义变量，不使用媒体查询方式，而是使用自定义属性（不用类名）
2. JS 检测暗黑模式和点击切换添加/删除自定义属性
3. localStorage 持久化主题，避免刷新后失效

```css
:root {
  --primary-bg: #eee;
  --primary-fg: #000;
  --secondary-bg: #ddd;
  --secondary-fg: #555;
  --primary-btn-bg: #000;
  --primary-btn-fg: #fff;
  --secondary-btn-bg: #ff0000;
  --secondary-btn-fg: #ffff00;
}
:root[data-theme="dark"] {
  --primary-bg: #282c35;
  --primary-fg: #fff;
  --secondary-bg: #1e2129;
  --secondary-fg: #aaa;
  --primary-btn-bg: #ddd;
  --primary-btn-fg: #222;
  --secondary-btn-bg: #780404;
  --secondary-btn-fg: #baba6a;
}
```

通过监听添加自定义属性 `data-theme`

```js
/* 刚进入页面先获取当前主题 */
if (!localStorage.getItem('theme')) {
  localStorage.setItem('theme', 'light')
}
document.documentElement.dataset.theme = localStorage.getItem('theme')
/* 监听系统的暗黑模式 */
const darkModeMediaQuery = window.matchMedia('(prefers-color-scheme: dark)')
function autoChange (e) {
  document.documentElement.dataset.theme = e.matches ? 'dark' : 'light'
  localStorage.setItem('theme', document.documentElement.dataset.theme)
}
// 兼容处理
if (typeof darkModeMediaQuery.addEventListener === 'function') {
  darkModeMediaQuery.addEventListener('change', autoChange)
} else if (typeof darkModeMediaQuery.addListener === 'function') {
  darkModeMediaQuery.addListener(autoChange)
}
/* 点击触发事件 */
function toggleTheme() {
  document.documentElement.dataset.theme = localStorage.getItem('theme') === 'light' ? 'dark' : 'light'
  localStorage.setItem('theme', document.documentElement.dataset.theme)
}
const toggleButton = document.querySelector("#dark-mode-toggle")
toggleButton.addEventListener('click', toggleTheme)
```

## 文字横向无限滚动

```html
<div id="wrap">
  <p class="text">Lorem ipsum dolor sit amet, consectetur adipisicing elit.</p>
</div>
<style>
#wrap {
  display: flex;
  box-sizing: border-box;
  width: 100%;
  text-wrap: nowrap;
  overflow: hidden;
  text-align: center;
}

.text {
  flex-shrink: 0;
  min-width: 100%;
  animation: none; /* 禁用初始动画 */
}
</style>
<script>
function startHorizontalScroll(element) {
  const GAP = 3
  const parentElement = element.parentNode
  const parentWidth = parentElement ? parentElement.getBoundingClientRect().width : window.innerWidth
  const textWidth = element.getBoundingClientRect().width

  function animateScroll(element, scrollX) {
    const duration = scrollX / 50

    element.animate(
      [
        { transform: `translateX(0px)` },
        { transform: `translateX(-${scrollX}px)` },
      ],
      {
        duration: duration * 1000,
        iterations: Number.POSITIVE_INFINITY,
      },
    )
  }

  if (textWidth > parentWidth) {
    const cloneElement = element.cloneNode(true)
    const scrollX = textWidth + GAP * Number.parseFloat(window.getComputedStyle(parentElement).fontSize)
    parentElement.appendChild(cloneElement)
    if (parentElement instanceof HTMLElement)
      parentElement.style.gap = `${GAP}em`

    animateScroll(element, scrollX)
    animateScroll(cloneElement, scrollX)
  }
}

// 获取所有带有 .text 类的元素，并依次应用滚动效果
const textElements = wrap.querySelectorAll('#wrap .text')
textElements.forEach(startHorizontalScroll)
</script>
```

<https://js.jirengu.com/jadelayuqe/1/edit?html,js,output>

## base 标签

```js
var baseElement = document.createElement('base');
// 设置 target 属性为 "_blank"
baseElement.setAttribute('target', '_blank');
// 将 base 元素插入到页面的 head 元素中
document.head.appendChild(baseElement);
```

## 网络状态监听

navigator.connection、navigator.onLine

online、offline
