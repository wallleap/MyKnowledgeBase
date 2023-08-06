---
title: 原子化 CSS 引擎 UnoCSS
date: 2023-08-04 11:29
updated: 2023-08-04 11:40
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - 原子化 CSS 引擎 UnoCSS
rating: 1
tags:
  - CSS
  - web
category: web
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
description: 文章描述
source: #
url: //myblog.wallleap.cn/post/1
---

## 前言

原子化 CSS(Atomic CSS）是近年来提出的一种架构方式，与之有区别的是我们比较常用的组件化书写方式。今天介绍的是 AntFu 的开源新作原子化 CSS 引擎—UnoCSS。

AntFu 在开发 Vitesse 是使用 [Tailwind CSS](https://link.juejin.cn/?target=https%3A%2F%2Ftailwindcss.com%2F "https://tailwindcss.com/") 作为 Vitesse 的默认 UI 框架。但由于 Tailwind 生成了数 MB 的 CSS，使得加载与更新 CSS 成为了整个 Vite 应用的性能瓶颈。后来作者发现了 [Windi CSS](https://link.juejin.cn/?target=https%3A%2F%2Fcn.windicss.org%2F "https://cn.windicss.org/")，相比于 [Tailwind CSS](https://link.juejin.cn/?target=https%3A%2F%2Ftailwindcss.com%2F "https://tailwindcss.com/") 具有按需加载，零依赖等特性。

由于作者是 vite 团队成员，所以该文案例所用的构建工具也将选用 vite

案例技术桟：Pnpm+Vite+Vue3

## 什么是原子化

John Polacek 在 [文章 Let’s Define Exactly What Atomic CSS is](https://link.juejin.cn/?target=https%3A%2F%2Fcss-tricks.com%2Flets-define-exactly-atomic-css%2F "https://css-tricks.com/lets-define-exactly-atomic-css/") 中有一个定义：

> Atomic CSS is the approach to CSS architecture that favors small, single-purpose classes with names based on visual function

译文：

> 原子化 CSS 是一种 CSS 的架构方式，它倾向于小巧且用途单一的 class，并且会以视觉效果进行命名。

单纯的定义并不够直观，你可以理解为下面的 CSS 书写方式就是原子化 CSS，仔细发现它的类名和样式值是有着强关联的。

这是一个简单的例子

```html
<div  class="m-0 text-red" /> 
```

```css
.m-0 {
  margin: 0;
}
.text-red {
  color: red;
}
/* ... */
```

## 原子化 vs 组件化

## 原子化

1. 减少了 CSS 体积，提高了 CSS 复用
2. 减少起名的复杂度
3. 增加了记忆成本 将 CSS 拆分为原子之后，你势必要记住一些 class 才能书写，哪怕 tailwindCSS 提供了完善的工具链，你写 background，也要记住开头是 bg
4. 可能会造成 class 名过长的问题

## 组件化

1. CSS 体积增加
2. 起名难

综上，原子化和组件化相结合似乎是更好的方向。

## 常见原子化 CSS 框架

在正式介绍 UnoCSS 之前，我们先来了解几个知名度高的同类思维 CSS 框架，热度比较高的有以下两种。

## [TailwindCSS](https://link.juejin.cn/?target=https%3A%2F%2Ftailwindcss.com%2F "https://tailwindcss.com/")

> 😃 持续迭代，目前已发布 3.0

知名度最高的一款 CSS 框架，2.0 版本增加了深色模式，JIT 引擎 (预先扫描源代码 ，按需编译所有 CSS)，相比传统的旧版本 Tailwind，保持了开发和生产环境的一致 (旧版本使用开发环境未处理，生成的 CSS 文件会有数 MB 的大小，生产环境使用了 [PurgCSS](https://link.juejin.cn/?target=https%3A%2F%2Fwww.tailwindcss.cn%2Fdocs%2Foptimizing-for-production%23purge-css-options "https://www.tailwindcss.cn/docs/optimizing-for-production#purge-css-options") 删除未使用的 CSS)

最新发布了 3.0 版本，默认开启 JIT 引擎 (2.0 需要手动开启)

## [Windi CSS](https://link.juejin.cn/?target=https%3A%2F%2Fcn.windicss.org%2F "https://cn.windicss.org/")

维护恐成难题

WindiCSS 是以 [TailwindCSS](https://link.juejin.cn/?target=https%3A%2F%2Fwww.tailwindcss.cn%2F "https://www.tailwindcss.cn/") 为灵感制作的库，并且兼容了 TailwindCSS，零依赖，也不要求用户安装 PostCSS 和 Autoprefixer，提供了更快的加载时间和热更新，同样支持按需生成，AntFu 大佬在 2021 年的测试结果如下，HMR 有 100 倍的效果。值得一提的是，WindCSS 增加了属性模式，这意味着你可以以写属性的形式去写 class 名。

![](https://cdn.wallleap.cn/img/pic/illustration/202308041130203.png)

## UnoCSS

Windi CSS 已经足够优秀，但 Antfu 仍旧没有满意，框架预设外的自定义工具的额外配置仍旧比较繁琐，不够灵活；所以他重新构想了原子化 CSS，UnoCSS 得以诞生。

> [UnoCSS](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fantfu%2Funocss "https://github.com/antfu/unocss") - 具有高性能且极具灵活性的即时原子化 CSS 引擎。

之所以把它称作引擎，是因为没有像 TailwindCSS，WindI CSS 那样提供核心的应用程序，所有能力由预设提供，突出核心：按需使用。

## 关于作者

知名开源作者 AntFu [Github](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fantfu "https://github.com/antfu")，Vite，Nuxt 团队核心人员，Slidev 作者

![](https://cdn.wallleap.cn/img/pic/illustration/202308041130204.png)

## 预设 (presets)

unoCSS 官方默认提供了三种预设

1. @unoCSS/preset-uno 工具类预设
2. @unoCSS/preset-attributify 属性化预设
3. @unoCSS/preset-icons 图标类 icon 支持

安装一波~😎

```sh
pnpm i -D unoCSS @unoCSS/preset-uno @unoCSS/preset-attributify @unoCSS/preset-icons
```

在 vite.config.ts 中引入，unoCSS 作为插件在 vite 中使用

```ts
// vite.config.ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import UnoCSS from 'unoCSS/vite'
import { presetUno, presetAttributify, presetIcons } from 'unoCSS'

import transformerDirective from "@unoCSS/transformer-directives";

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [vue(),
  UnoCSS({
    presets: [presetUno(), presetAttributify(), presetIcons()],
    transformers: [transformerDirective()],
  })],
})
```

安装 vscode 插件，获得良好的输入提示。

![](https://cdn.wallleap.cn/img/pic/illustration/202308041130205.png)

### @unoCSS/preset-uno

这个预设提供了流行的实用程序优先框架的通用超集，包括 Tailwind CSS，Windi CSS，Bootstrap，Tachyons 等

比如 `ml-3`（Tailwind），`ms-2`（Bootstrap），`ma4`（Tachyons），`mt-10px`（Windi CSS）

这些写法都会生效

```html
<div w-20 h-20 bg-blue  ml-3 ms-2 ma4 mt-10px />
```

### @unoCSS/preset-attributify

继承了 WindiCSS 的属性化模式，简化了书写 class，以属性的形式去写 class，但是在使用组件的时候，较大可能出现属性太多，容易混淆的情况。

```html
<button
  bg="blue-400 hover:blue-500 dark:blue-500 dark:hover:blue-600"
  text="sm white"
>
  Click
</button>
```

### @unoCSS/preset-icons

UnoCSS 提供了图标的预设，它是纯 CSS 的图标，可以选择 [Icones](https://link.juejin.cn/?target=https%3A%2F%2Ficones.js.org%2F "https://icones.js.org/") 或 [Iconify](https://link.juejin.cn/?target=https%3A%2F%2Ficon-sets.iconify.design%2F "https://icon-sets.iconify.design/") 作为图标源使用，同样也支持自定义 icon，本身实现按需加载。

1. 安装

```sh
# 安装全部，大概140多M，开发时会安装，打包只会使用用到的图标
pnpm i -D @unoCSS/preset-icons @iconify-json

# 安装某个图标集合
pnpm i -D @unoCSS/preset-icons @iconify-json/icon集合id(例如icon-park-twotone）
```

[icones.js.org/collection/…](https://link.juejin.cn/?target=https%3A%2F%2Ficones.js.org%2Fcollection%2Ficon-park-twotone "https://icones.js.org/collection/icon-park-twotone") 中的 icon-park-twotone 即为这个 Icon 集合 id

![](https://cdn.wallleap.cn/img/pic/illustration/202308041130206.png)

使用

以 [Icones](https://link.juejin.cn/?target=https%3A%2F%2Ficones.js.org%2F "https://icones.js.org/") 集合为例，复制 i 开头或者点击 UnoCSS 得到类名。

![](https://cdn.wallleap.cn/img/pic/illustration/202308041130208.png)

```html
<div class="i-icon-park-twotone-winking-face"> </div>
```

多色图标

直接设置 color 值即可，属性化模式示例，color-blue

```html
<div i-icon-park-twotone-winking-face colo text-size-100px  color-blue> </div>
```

额外属性

可以提供额外的 CSS 属性来控制图标的默认行为，默认情况下图标内联示例:

```ts
// vite.config.ts

presetIcons({
  extraProperties: {
    'display': 'inline-block',
    'vertical-align': 'middle',
    // ...
  },
})
```

更多查看 [github.com/unoCSS/unoC…](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FunoCSS%2FunoCSS%2Ftree%2Fmain%2Fpackages%2Fpreset-icons "https://github.com/unoCSS/unoCSS/tree/main/packages/preset-icons")

## 插件 (plugins)

### @unoCSS/transformer-directives

- 官方提供了这个插件实现在 style 中使用 apply 指令写原子化 CSS

```js
...
export default defineConfig({
  plugins: [
    vue(),
    UnoCSS({
      ...
      transformers: [transformerDirective()]
    })
  ]
});
```

## 规则 (rules)

除了官方默认的工具类外，支持自定义 CSS 规则，配置 rules 数组即可，支持方式 1 和方式 2(正则匹配) 两种。

```ts
// vite.config.ts
export default defineConfig({
  plugins: [
    vue(),
    UnoCSS({
      ...
      rules: [
        // 方式1
        [ 
          "p-a", 
          {  
            position: "absolute",
          }
        ],
        // 方式2
        [/^m-(\d+)$/, ([, d]) => ({ margin: `${d / 4}px` })]
      ],

    })
  ]
});
```

项目中使用时会解析成对应的 CSS 样式，`<div class="m-20"></div>` 对应的 style 为

```css
.m-20{
  margin : 5px;
}
```

值得一提，在安装了 UnoCSS 提供的 VsCode 插件后，这些自定义的部分同样会提示出来。

## 纯 CSS 图标

顾名思义，不含有任何 js 元素，纯 CSS 实现。

## 现有方案

有个名为 [css.gg](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fastrit%2Fcss.gg "https://github.com/astrit/css.gg") 的纯 CSS 图标解决方案，它完全通过伪元素（`::before`，`::after`）来构建图标，使用这种方案需要对 CSS 工作原理有着深刻理解。

## UnoCSS 中的方案

另一种方案，是将 svg 转换成 dataurl，AntFu 这里使用了 Base64，并做了一系列优化，也就是说我们看到的图标其实是图片，配合 CSS 的 mask 属性可以轻松实现多色图标。

![](https://cdn.wallleap.cn/img/pic/illustration/202308041130209.png)

处理 svg 字符串，转为 **dataurl**

```js
const dataUrl = `data:image/svg+xml;base64,${Buffer.from(svg).toString('base64')}`
```

svg 本身是文本格式，转为 Base64 会变大，因此需要优化大小。

优化 - **减小体积**

```js
// https://bl.ocks.org/jennyknuth/222825e315d45a738ed9d6e04c7a88d0
// https://codepen.io/tigt/post/optimizing-svgs-in-data-uris

// 编码器
function encodeSvg(svg: string) {
    return svg.replace('<svg', (~svg.indexOf('xmlns') ? '<svg' : '<svg xmlns="http://www.w3.org/2000/svg"'))
      .replace(/"/g, '\'')
      .replace(/%/g, '%25')
      .replace(/#/g, '%23')
      .replace(/{/g, '%7B')
      .replace(/}/g, '%7D')
      .replace(/</g, '%3C')
      .replace(/>/g, '%3E')
  }
  
  const dataUrl = `data:image/svg+xml;utf8,${encodeSvg(svg)}`       
```

**输出结果**：传入 svg 字符串运行 encodesvg 方法我们会得到如下格式的 url，最小编码的 url 诞生。

```
%3Csvg%20xmlns%3D%22http%3A%2F%2Fwww.w3.org%2F2000%2Fsvg%22%20viewBox%3D%220%200%20512%20512%22%3E%3Cpath%20d%3D%22M224%20387.814V512L32%20320l192-192v126.912C447.375%20260.152%20437.794%20103.016%20380.93%200%20521.287%20151.707%20491.48%20394.785%20224%20387.814z%22%2F%3E%3C%2Fsvg%3E
```

## 写在最后

对比 TailWindCSS，UnoCSS 可能显得并不是很出类拔萃，但它有着更小，更灵活的优势；对一个开源项目来讲，开坑并不难，难的是维护，对比 TailWindCSS 一个公司级维护的项目，UnoCSS 仍旧有着很长的路要走。

## 参考资料

[重新构想原子化CSS](https://link.juejin.cn/?target=https%3A%2F%2Fantfu.me%2Fposts%2Freimagine-atomic-css-zh "https://antfu.me/posts/reimagine-atomic-css-zh")
