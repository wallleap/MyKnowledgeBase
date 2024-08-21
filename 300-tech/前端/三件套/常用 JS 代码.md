---
title: 常用 JS 代码
date: 2023-08-01T02:52:00+08:00
updated: 2024-08-21T10:32:36+08:00
dg-publish: false
aliases:
  - 常用 JS 代码
author: Luwang
category: 分类
comments: true
cover: //cdn.wallleap.cn/img/post/1.jpg
description: 文章描述
image-auto-upload: true
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
rating: 1
source: #
tags:
  - blog
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

## Transition View 暗黑切换动画

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  <style>
    :root {
      --black: #333333;
      --white: #f5f5f5;
      --background: var(--white);
      --foreground: var(--black);
    }

    :root[data-theme="dark"] {
      --background: var(--black);
      --foreground: var(--white);
    }

    html {
      background: var(--background);
      color: var(--foreground);
    }

    html,
    body {
      height: 100%;
    }

    .toggle {
      position: absolute;
      cursor: pointer;
      top: 20px;
      right: 25px;
      font-size: 150%;
    }

    .text {
      width: 100%;
      height: 100%;
      display: flex;
      justify-content: center;
      align-items: center;
    }

    /**
      Animated Theme Toggle
    */
    ::view-transition-old(root),
    ::view-transition-new(root) {
      animation: none;
      mix-blend-mode: normal;
    }

    [data-theme="dark"]::view-transition-old(root) {
      z-index: 1;
    }

    [data-theme="dark"]::view-transition-new(root) {
      z-index: 999;
    }

    ::view-transition-old(root) {
      z-index: 999;
    }

    ::view-transition-new(root) {
      z-index: 1;
    }
  </style>
</head>
<body>
  <div class="text">Text</div>
  <span class="toggle">亮</span>
  <script>
    /**
     * 切换主题色，扩散渐变动画
     * @param {MouseEvent} event 点击事件
     */
    function toggleTheme(event) {
      const willDark = !isDark()
      event.target.innerText = willDark ? "暗" : "亮"
      // 浏览器新特性不支持 或者 开启了动画减弱
      if (!document.startViewTransition || isReducedMotion()) {
        toggleDark();
        return;
      }

      const transition = document.startViewTransition(() => {
        toggleDark();
      });

      // 传入点击事件，从点击处开始扩散。否则，从右上角开始扩散
      const x = event?.clientX ?? window.innerWidth;
      const y = event?.clientY ?? 0;

      const endRadius = Math.hypot(
        Math.max(x, innerWidth - x),
        Math.max(y, innerHeight - y)
      );
      void transition.ready.then(() => {
        const clipPath = [
          `circle(0px at ${x}px ${y}px)`,
          `circle(${endRadius}px at ${x}px ${y}px)`
        ];
        document.documentElement.animate({
          clipPath: willDark ? clipPath : [...clipPath].reverse()
        }, {
          duration: 500,
          easing: "ease-in",
          pseudoElement: willDark ?
            "::view-transition-new(root)" :
            "::view-transition-old(root)"
        });
      });
    }
    /**
     * 切换主题色，html标签切换dark类
     */
    function toggleDark() {
      document.documentElement.dataset.theme = document.documentElement.dataset.theme === "dark" ? "light" : "dark";
    }

    /**
     * 检测用户的系统是否被开启了动画减弱功能
     * @link https://developer.mozilla.org/zh-CN/docs/Web/CSS/@media/prefers-reduced-motion
     */
    function isReducedMotion() {
      return window.matchMedia(`(prefers-reduced-motion: reduce)`).matches === true;
    }

    /**
     * 当前主题色是否是暗色
     */
    function isDark() {
      return document.documentElement.dataset.theme === 'dark'
    }
    
    document.querySelector(".toggle").addEventListener("click", function (event) {
      toggleTheme(event)
    })
  </script>
</body>
</html>
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
